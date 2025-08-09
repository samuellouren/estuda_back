// ===========================================
// CRUD COMPLETO - TODOS OS MÉTODOS HTTP
// ===========================================
/*
CRUD = Create, Read, Update, Delete
POST   = Create (Criar)
GET    = Read (Ler/Buscar)  
PUT    = Update (Atualizar)
DELETE = Delete (Apagar)
*/

const express = require('express');
const app = express();

// Middleware para entender JSON
app.use(express.json());

// ===========================================
// "BANCO DE DADOS" SIMULADO
// ===========================================
let usuarios = [
  { id: 1, nome: "João Silva", email: "joao@email.com", tipo: "estudante" },
  { id: 2, nome: "Maria Santos", email: "maria@email.com", tipo: "instrutor" },
  { id: 3, nome: "Pedro Costa", email: "pedro@email.com", tipo: "estudante" }
];

// Variável para gerar IDs únicos
let proximoId = 4;

// ===========================================
// 1. GET - BUSCAR/LER DADOS
// ===========================================

// GET todos os usuários
app.get('/usuarios', (req, res) => {
  console.log('📖 GET: Buscando todos os usuários');
  
  res.json({
    total: usuarios.length,
    usuarios: usuarios
  });
});

// GET um usuário específico
app.get('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  
  console.log(`📖 GET: Buscando usuário com ID ${id}`);
  
  // Procura o usuário
  const usuario = usuarios.find(u => u.id === id);
  
  // Se não encontrou
  if (!usuario) {
    return res.status(404).json({ 
      erro: 'Usuário não encontrado' 
    });
  }
  
  // Se encontrou
  res.json(usuario);
});

// GET com filtro por tipo
app.get('/usuarios/tipo/:tipo', (req, res) => {
  const tipo = req.params.tipo;
  
  console.log(`📖 GET: Buscando usuários do tipo ${tipo}`);
  
  const usuariosFiltrados = usuarios.filter(u => u.tipo === tipo);
  
  res.json({
    tipo: tipo,
    total: usuariosFiltrados.length,
    usuarios: usuariosFiltrados
  });
});

// ===========================================
// 2. POST - CRIAR NOVOS DADOS
// ===========================================

// POST - Criar novo usuário
app.post('/usuarios', (req, res) => {
  console.log('✨ POST: Criando novo usuário');
  console.log('Dados recebidos:', req.body);
  
  // Pega os dados enviados no corpo da requisição
  const { nome, email, tipo } = req.body;
  
  // Validações básicas
  if (!nome) {
    return res.status(400).json({ erro: 'Nome é obrigatório' });
  }
  
  if (!email) {
    return res.status(400).json({ erro: 'Email é obrigatório' });
  }
  
  if (!tipo || (tipo !== 'estudante' && tipo !== 'instrutor')) {
    return res.status(400).json({ 
      erro: 'Tipo deve ser "estudante" ou "instrutor"' 
    });
  }
  
  // Verifica se email já existe
  const emailJaExiste = usuarios.find(u => u.email === email);
  if (emailJaExiste) {
    return res.status(400).json({ erro: 'Email já está em uso' });
  }
  
  // Cria o novo usuário
  const novoUsuario = {
    id: proximoId++,
    nome: nome,
    email: email,
    tipo: tipo
  };
  
  // Adiciona na lista
  usuarios.push(novoUsuario);
  
  console.log('✅ Usuário criado:', novoUsuario);
  
  // Responde com status 201 (Created)
  res.status(201).json({
    mensagem: 'Usuário criado com sucesso!',
    usuario: novoUsuario
  });
});

// ===========================================
// 3. PUT - ATUALIZAR DADOS COMPLETOS
// ===========================================

// PUT - Atualizar usuário completo
app.put('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const { nome, email, tipo } = req.body;
  
  console.log(`🔄 PUT: Atualizando usuário ${id}`);
  console.log('Novos dados:', req.body);
  
  // Encontra o usuário
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
  // Validações (mesmas do POST)
  if (!nome) {
    return res.status(400).json({ erro: 'Nome é obrigatório' });
  }
  
  if (!email) {
    return res.status(400).json({ erro: 'Email é obrigatório' });
  }
  
  if (!tipo || (tipo !== 'estudante' && tipo !== 'instrutor')) {
    return res.status(400).json({ 
      erro: 'Tipo deve ser "estudante" ou "instrutor"' 
    });
  }
  
  // Verifica se email já existe em outro usuário
  const emailJaExiste = usuarios.find(u => u.email === email && u.id !== id);
  if (emailJaExiste) {
    return res.status(400).json({ erro: 'Email já está em uso' });
  }
  
  // Salva dados antigos para log
  const dadosAntigos = { ...usuarios[indiceUsuario] };
  
  // Atualiza TODOS os dados (PUT substitui completamente)
  usuarios[indiceUsuario] = {
    id: id, // mantém o ID
    nome: nome,
    email: email,
    tipo: tipo
  };
  
  console.log('📊 Dados antigos:', dadosAntigos);
  console.log('✅ Dados novos:', usuarios[indiceUsuario]);
  
  res.json({
    mensagem: 'Usuário atualizado com sucesso!',
    usuario: usuarios[indiceUsuario]
  });
});

// ===========================================
// 4. PATCH - ATUALIZAR DADOS PARCIAIS
// ===========================================

// PATCH - Atualizar apenas alguns campos
app.patch('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const atualizacoes = req.body;
  
  console.log(`🔧 PATCH: Atualizando parcialmente usuário ${id}`);
  console.log('Campos a atualizar:', atualizacoes);
  
  // Encontra o usuário
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
  // Validações apenas para campos que foram enviados
  if (atualizacoes.tipo && atualizacoes.tipo !== 'estudante' && atualizacoes.tipo !== 'instrutor') {
    return res.status(400).json({ 
      erro: 'Tipo deve ser "estudante" ou "instrutor"' 
    });
  }
  
  if (atualizacoes.email) {
    const emailJaExiste = usuarios.find(u => u.email === atualizacoes.email && u.id !== id);
    if (emailJaExiste) {
      return res.status(400).json({ erro: 'Email já está em uso' });
    }
  }
  
  // Salva dados antigos
  const dadosAntigos = { ...usuarios[indiceUsuario] };
  
  // Atualiza apenas os campos enviados
  usuarios[indiceUsuario] = {
    ...usuarios[indiceUsuario], // mantém dados existentes
    ...atualizacoes // sobrescreve apenas o que foi enviado
  };
  
  console.log('📊 Dados antigos:', dadosAntigos);
  console.log('✅ Dados atualizados:', usuarios[indiceUsuario]);
  
  res.json({
    mensagem: 'Usuário atualizado parcialmente!',
    usuario: usuarios[indiceUsuario]
  });
});

// ===========================================
// 5. DELETE - APAGAR DADOS
// ===========================================

// DELETE - Apagar usuário
app.delete('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  
  console.log(`🗑️  DELETE: Removendo usuário ${id}`);
  
  // Encontra o usuário
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
  // Salva dados do usuário que será removido
  const usuarioRemovido = usuarios[indiceUsuario];
  
  // Remove da lista
  usuarios.splice(indiceUsuario, 1);
  
  console.log('✅ Usuário removido:', usuarioRemovido);
  console.log(`📊 Total de usuários agora: ${usuarios.length}`);
  
  res.json({
    mensagem: 'Usuário removido com sucesso!',
    usuarioRemovido: usuarioRemovido,
    totalRestante: usuarios.length
  });
});

// DELETE - Apagar todos os usuários de um tipo
app.delete('/usuarios/tipo/:tipo', (req, res) => {
  const tipo = req.params.tipo;
  
  console.log(`🗑️  DELETE: Removendo todos os usuários do tipo ${tipo}`);
  
  // Conta quantos serão removidos
  const usuariosParaRemover = usuarios.filter(u => u.tipo === tipo);
  const totalRemover = usuariosParaRemover.length;
  
  if (totalRemover === 0) {
    return res.status(404).json({ 
      erro: `Nenhum usuário do tipo "${tipo}" encontrado` 
    });
  }
  
  // Remove todos do tipo especificado
  usuarios = usuarios.filter(u => u.tipo !== tipo);
  
  console.log(`✅ ${totalRemover} usuários removidos`);
  
  res.json({
    mensagem: `${totalRemover} usuários do tipo "${tipo}" foram removidos`,
    usuariosRemovidos: usuariosParaRemover,
    totalRestante: usuarios.length
  });
});

// ===========================================
// 6. ROTAS DE AJUDA E TESTE
// ===========================================

// GET - Ver estado atual dos dados
app.get('/', (req, res) => {
  res.json({
    mensagem: 'API de Usuários - CRUD Completo',
    totalUsuarios: usuarios.length,
    proximoId: proximoId,
    usuarios: usuarios,
    endpoints: {
      'GET /usuarios': 'Listar todos os usuários',
      'GET /usuarios/:id': 'Buscar usuário por ID',
      'GET /usuarios/tipo/:tipo': 'Buscar usuários por tipo',
      'POST /usuarios': 'Criar novo usuário',
      'PUT /usuarios/:id': 'Atualizar usuário completo',
      'PATCH /usuarios/:id': 'Atualizar usuário parcialmente',
      'DELETE /usuarios/:id': 'Remover usuário',
      'DELETE /usuarios/tipo/:tipo': 'Remover usuários por tipo'
    }
  });
});

// ===========================================
// 7. INICIAR SERVIDOR
// ===========================================

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`
🚀 Servidor CRUD rodando na porta ${PORT}
📍 Acesse: http://localhost:${PORT}

📋 COMO TESTAR:

1. VER DADOS:
   GET http://localhost:${PORT}/

2. BUSCAR:
   GET http://localhost:${PORT}/usuarios
   GET http://localhost:${PORT}/usuarios/1
   GET http://localhost:${PORT}/usuarios/tipo/estudante

3. CRIAR (use Postman/Insomnia):
   POST http://localhost:${PORT}/usuarios
   Body: {
     "nome": "Ana Silva",
     "email": "ana@email.com", 
     "tipo": "estudante"
   }

4. ATUALIZAR COMPLETO:
   PUT http://localhost:${PORT}/usuarios/1
   Body: {
     "nome": "João Santos",
     "email": "joao.novo@email.com",
     "tipo": "instrutor"
   }

5. ATUALIZAR PARCIAL:
   PATCH http://localhost:${PORT}/usuarios/1
   Body: {
     "nome": "João Oliveira"
   }

6. REMOVER:
   DELETE http://localhost:${PORT}/usuarios/1
   DELETE http://localhost:${PORT}/usuarios/tipo/estudante
  `);
});

// ===========================================
// CÓDIGOS DE STATUS HTTP MAIS COMUNS:
// ===========================================
/*
200 - OK (sucesso)
201 - Created (criado com sucesso)
400 - Bad Request (erro na requisição)
404 - Not Found (não encontrado)
500 - Internal Server Error (erro do servidor)
*/
