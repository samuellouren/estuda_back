const express = require('express');
const app = express();

app.use(express.json());

let usuarios = [
  { id: 1, nome: "João Silva", email: "joao@email.com", tipo: "estudante" },
  { id: 2, nome: "Maria Santos", email: "maria@email.com", tipo: "instrutor" },
  { id: 3, nome: "Pedro Costa", email: "pedro@email.com", tipo: "estudante" }
];

let proximoId = 4;

app.get('/usuarios', (req, res) => {
  console.log('GET: Buscando todos os usuários');
  
  res.json({
    total: usuarios.length,
    usuarios: usuarios
  });
});

app.get('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  
  console.log(`GET: Buscando usuário com ID ${id}`);
  
  const usuario = usuarios.find(u => u.id === id);
  
  if (!usuario) {
    return res.status(404).json({ 
      erro: 'Usuário não encontrado' 
    });
  }
  
  res.json(usuario);
});

app.get('/usuarios/tipo/:tipo', (req, res) => {
  const tipo = req.params.tipo;
  
  console.log(`GET: Buscando usuários do tipo ${tipo}`);
  
  const usuariosFiltrados = usuarios.filter(u => u.tipo === tipo);
  
  res.json({
    tipo: tipo,
    total: usuariosFiltrados.length,
    usuarios: usuariosFiltrados
  });
});

app.post('/usuarios', (req, res) => {
  console.log('POST: Criando novo usuário');
  console.log('Dados recebidos:', req.body);
  
  const { nome, email, tipo } = req.body;
  
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
  
  const emailJaExiste = usuarios.find(u => u.email === email);
  if (emailJaExiste) {
    return res.status(400).json({ erro: 'Email já está em uso' });
  }
  
  const novoUsuario = {
    id: proximoId++,
    nome: nome,
    email: email,
    tipo: tipo
  };
  
  usuarios.push(novoUsuario);
  
  console.log('Usuário criado:', novoUsuario);
  
  res.status(201).json({
    mensagem: 'Usuário criado com sucesso!',
    usuario: novoUsuario
  });
});

app.put('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const { nome, email, tipo } = req.body;
  
  console.log(`PUT: Atualizando usuário ${id}`);
  console.log('Novos dados:', req.body);
  
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
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
  
  const emailJaExiste = usuarios.find(u => u.email === email && u.id !== id);
  if (emailJaExiste) {
    return res.status(400).json({ erro: 'Email já está em uso' });
  }
  
  const dadosAntigos = { ...usuarios[indiceUsuario] };
  
  usuarios[indiceUsuario] = {
    id: id,
    nome: nome,
    email: email,
    tipo: tipo
  };
  
  console.log('Dados antigos:', dadosAntigos);
  console.log('Dados novos:', usuarios[indiceUsuario]);
  
  res.json({
    mensagem: 'Usuário atualizado com sucesso!',
    usuario: usuarios[indiceUsuario]
  });
});

app.patch('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const atualizacoes = req.body;
  
  console.log(`PATCH: Atualizando parcialmente usuário ${id}`);
  console.log('Campos a atualizar:', atualizacoes);
  
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
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
  
  const dadosAntigos = { ...usuarios[indiceUsuario] };
  
  usuarios[indiceUsuario] = {
    ...usuarios[indiceUsuario],
    ...atualizacoes
  };
  
  console.log('Dados antigos:', dadosAntigos);
  console.log('Dados atualizados:', usuarios[indiceUsuario]);
  
  res.json({
    mensagem: 'Usuário atualizado parcialmente!',
    usuario: usuarios[indiceUsuario]
  });
});

app.delete('/usuarios/:id', (req, res) => {
  const id = parseInt(req.params.id);
  
  console.log(`DELETE: Removendo usuário ${id}`);
  
  const indiceUsuario = usuarios.findIndex(u => u.id === id);
  
  if (indiceUsuario === -1) {
    return res.status(404).json({ erro: 'Usuário não encontrado' });
  }
  
  const usuarioRemovido = usuarios[indiceUsuario];
  
  usuarios.splice(indiceUsuario, 1);
  
  console.log('Usuário removido:', usuarioRemovido);
  console.log(`Total de usuários agora: ${usuarios.length}`);
  
  res.json({
    mensagem: 'Usuário removido com sucesso!',
    usuarioRemovido: usuarioRemovido,
    totalRestante: usuarios.length
  });
});

app.delete('/usuarios/tipo/:tipo', (req, res) => {
  const tipo = req.params.tipo;
  
  console.log(`DELETE: Removendo todos os usuários do tipo ${tipo}`);
  
  const usuariosParaRemover = usuarios.filter(u => u.tipo === tipo);
  const totalRemover = usuariosParaRemover.length;
  
  if (totalRemover === 0) {
    return res.status(404).json({ 
      erro: `Nenhum usuário do tipo "${tipo}" encontrado` 
    });
  }
  
  usuarios = usuarios.filter(u => u.tipo !== tipo);
  
  console.log(`${totalRemover} usuários removidos`);
  
  res.json({
    mensagem: `${totalRemover} usuários do tipo "${tipo}" foram removidos`,
    usuariosRemovidos: usuariosParaRemover,
    totalRestante: usuarios.length
  });
});

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

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
