# TODO-list
Trabalho 2
Lista de tarefas

Lista de Tarefas com PHP, PDO e AJAX Este é um projeto simples de lista de tarefas (To-Do List) que demonstra o uso de PHP com PDO para manipulação do banco de dados, e AJAX com jQuery para operações assíncronas (CRUD), oferecendo uma experiência de usuário moderna sem recarregar uma página.

Funcionalidades Adicionar novas tarefas (título e descrição opcionais).

Listar todas as tarefas, ordenadas por status (não concluídas primeiro).

Marcar/Desmarcar tarefas como concluídas com um clique.

Excluir tarefas.

Interface responsiva e moderna utilizando Bootstrap 5 e ícones Font Awesome.

Utilize SweetAlert2 para notificações amigáveis ao usuário.

Tecnologias Utilizadas Backend: PHP

Banco de Dados: MySQL

Driver de BD: PHP Data Objects (PDO)

Front-end: HTML5, CSS3, JavaScript

Bibliotecas:

Bootstrap 5

jQuery (para AJAX)

Fonte incrível

SweetAlert2

Banco de Dados- XAMPP todolist_db.sql -- phpMyAdmin SQL Dump -- versão 5.2.1 -- https://www.phpmyadmin.net/
-- Host: 127.0.0.1 -- Tempo de geração: 28/09/2025 às 17h33 -- Versão do servidor: 10.4.32-MariaDB -- Versão do PHP: 8.2.12

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO"; INICIAR TRANSAÇÃO; SET time_zone = "+00:00";

/*!40101 DEFINIR @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT /; / !40101 DEFINIR @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS /; / !40101 DEFINIR @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION /; / !40101 DEFINIR NOMES utf8mb4 */;

-- -- Banco de dados:todolist_db
-- -- Estrutura para tabelatarefas
CRIAR TABELA tarefas( idint(11) NÃO NULO, titulovarchar(255) NÃO NULO, descricaotexto PADRÃO NULO, concluidatinyint(1) NÃO NULO PADRÃO 0, data_criacaoregistro de data e hora NÃO NULO PADRÃO current_timestamp() ) ENGINE=InnoDB PADRÃO CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

-- -- Índices para tabelas descartadas
-- -- Índices de tabelatarefas
ALTER TABLE tarefas ADD PRIMARY KEY ( id);

-- -- AUTO_INCREMENT para tabelas descartadas
-- -- AUTO_INCREMENT da tabelatarefas
ALTER TABLE tarefas MODIFY idint(11) NÃO NULO INCREMENTO_AUTO; CONFIRMAÇÃO;

/*!40101 DEFINIR CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT /; / !40101 DEFINIR CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS /; / !40101 DEFINIR COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;

Configuração PHP-config.php

PDO::ERRMODE_EXCEPTION, PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC, PDO::ATTR_EMULATE_PREPARES => false, ]; try { return new PDO($dsn, DB_USER, DB_PASS, $options); } catch (\PDOException $e) { die("Erro de Conexão com o Banco de Dados: " . $e->getMessage()); } } $pdo = connectDB(); if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['action'])) { header('Content-Type: application/json'); $response = ['success' => false, 'message' => '']; tente { switch ($_POST['action']) { case 'add': $titulo = trim($_POST['titulo']); $descrição = trim($_POST['descrição']); if (empty($titulo)) { throw new Exception("O título da tarefa não pode ser vazio."); } $stmt = $pdo->prepare("INSERT INTO tarefas (título, descrição) VALUES (?, ?)"); $stmt->executar([$titulo, $descricao]); $resposta['sucesso'] = verdadeiro; $response['message'] = "Tarefa selecionada com sucesso!"; quebrar; caso 'excluir': $id = (int)$_POST['id']; $stmt = $pdo->prepare("DELETE FROM tarefas WHERE id = ?"); $stmt->executar([$id]); $response['success'] = true; $response['message'] = "Tarefa excluída com sucesso!"; break; case 'toggle_status': $id = (int)$_POST['id']; $stmt = $pdo->prepare("UPDATE tarefas SET concluída = 1 - concluída WHERE id = ?"); $stmt->execute([$id]); $stmt_status = $pdo->prepare("SELECT concluída FROM tarefas WHERE id = ?"); $stmt_status->execute([$id]); $novo_status = $stmt_status->fetchColumn(); $response['success'] = true; $response['message'] = "Status da tarefa atualizada."; $response['novo_status'] = $novo_status; break; default: $response['message'] = "Ação inválida."; } } catch (Exceção $e) { $response['message'] = "Erro: " . $e->getMessage(); } echo json_encode($response); saída; } function getTarefas() { global $pdo; $stmt = $pdo->query("SELECT * FROM tarefas ORDER BY concluída ASC, data_criação DESC"); return $stmt->fetchAll(); } ?>
index.php

<title>Lista de Tarefas com PHP, PDO e AJAX</title> <style> body { background-color: #f4f6f9; } .card { box-shadow: 0 0 1px rgba(0,0,0,.125), 0 1px 3px rgba(0,0,0,.2); border-radius: .25rem; } .card-header { background-color: #ffffff; border-bottom: 1px solid rgba(0,0,0,.125); } .todo-list .form-check-label.done { text-decoration: line-through; color: #6c757d; } </style>
Lista de Tarefas
        <div class="card card-primary card-outline mb-4">
            <div class="card-header">
                <h3 class="card-title">Nova Tarefa</h3>
            </div>
            <div class="card-body">
                <form id="form-add-tarefa" class="mb-3">
                    <div class="mb-3">
                        <label for="titulo" class="form-label">Título</label>
                        <input type="text" class="form-control" id="titulo" name="titulo" required>
                    </div>
                    <div class="mb-3">
                        <label for="descricao" class="form-label">Descrição (Opcional)</label>
                        <textarea class="form-control" id="descricao" name="descricao" rows="2"></textarea>
                    </div>
                    <button type="submit" class="btn btn-primary float-end">
                        <i class="fas fa-plus"></i> Adicionar Tarefa
                    </button>
                </form>
            </div>
        </div>

        <div class="card" id="card-tarefas">
            <div class="card-header">
                <h3 class="card-title">Minhas Tarefas</h3>
            </div>
            <div class="card-body p-0">
                <ul class="todo-list list-group list-group-flush" id="lista-tarefas">
                    <?php if (count($tarefas) > 0): ?>
                        <?php foreach ($tarefas as $tarefa): ?>
                            <?php $is_done = $tarefa['concluida'] ? 'done' : ''; ?>
                            <li class="list-group-item d-flex align-items-center todo-item-<?= $tarefa['id'] ?> <?= $is_done ?>" data-id="<?= $tarefa['id'] ?>">
                                <div class="form-check me-3">
                                    <input class="form-check-input check-status" type="checkbox" data-id="<?= $tarefa['id'] ?>" <?= $tarefa['concluida'] ? 'checked' : '' ?>>
                                </div>
                                <div class="flex-grow-1">
                                    <p class="mb-0 fw-bold <?= $is_done ?>"><?= htmlspecialchars($tarefa['titulo']) ?></p>
                                    <?php if (!empty($tarefa['descricao'])): ?>
                                        <small class="text-muted <?= $is_done ?>"><?= nl2br(htmlspecialchars($tarefa['descricao'])) ?></small>
                                    <?php endif; ?>
                                </div>
                                <button type="button" class="btn btn-danger btn-sm delete-btn ms-3" data-id="<?= $tarefa['id'] ?>" title="Excluir">
                                    <i class="fas fa-trash"></i>
                                </button>
                            </li>
                        <?php endforeach; ?>
                    <?php else: ?>
                        <li class="list-group-item text-center text-muted" id="empty-list-message">
                            Nenhuma tarefa cadastrada.
                        </li>
                    <?php endif; ?>
                </ul>
            </div>
            <div class="card-footer clearfix">
                <p class="mb-0 text-muted float-end">Total de Tarefas: <span id="total-tarefas"><?= count($tarefas) ?></span></p>
            </div>
        </div>

    </div>
</div>
<script src=" https://code.jquery.com/jquery-3.6.0.min.js"></script> <script src=" https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script> <script src=" https://cdn.jsdelivr.net/npm/sweetalert2@11"></script> <script> $(document).ready(function() { $('#form-add-tarefa').on('submit', function(e) { e.preventDefault(); var titulo = $('#titulo').val().trim(); if (titulo === '') { Swal.fire('Atenção', 'O título da tarefa não pode ser vazio.', 'warning'); return; } $.ajax({ url: 'config.php', type: 'POST', dataType: 'json', data: { action: 'add', titulo: titulo, descricao: $('#descricao').val() }, sucesso: function(response) { if (response.success) { Swal.fire('Sucesso!', response.message, 'success'); localização.reload(); } else { Swal.fire('Erro!', resposta.message, 'erro'); } }, erro: function() { Swal.fire('Erro de Conexão', 'Não foi possível se comunicar com o servidor.', 'error'); } }); }); $(document).on('change', '.check-status', function() { var taskId = $(this).data('id'); var listItem = $('.todo-item-' + taskId); listItem.toggleClass('done'); listItem.find('.fw-bold, .text-muted').toggleClass('done'); $.ajax({ url: 'config.php', tipo: 'POST', dataType: 'json', dados: { ação: 'toggle_status', id: taskId }, sucesso: função (resposta) { if (response.success) { if (response.novo_status == 1) { $('#lista-tarefas').append(listItem); } else { var firstNotDone = $('#lista-tarefas > li:not(.done):first'); if(firstNotDone.length) { listItem.insertBefore(firstNotDone); } else { $('#lista-tarefas').prepend(listItem); } } Swal.fire('Atualizado!', response.message, 'success'); } else { listItem.toggleClass('done'); listItem.find('.fw-bold, .text-muted').toggleClass('done'); $(this).prop('checked', !$(this).prop('checked')); Swal.fire('Erro!', response.message, 'error'); } }, error:function() { listItem.toggleClass('concluído'); listItem.find('.fw-bold, .text-muted').toggleClass('concluído'); $(este).prop('verificado', !$(este).prop('verificado')); Swal.fire('Erro de Conexão', 'Não foi possível atualizar o status.', 'error'); } }); }); $(document).on('click', '.delete-btn', function() { var taskId = $(this).data('id'); var listItem = $('.todo-item-' + taskId); Swal.fire({ title: 'Tem certeza?', text: "Esta ação não pode ser desfeita!", icon: 'warning', showCancelButton: true, confirmButtonColor: '#d33', cancelButtonColor: '#3085d6', confirmButtonText: 'Sim, excluir!', cancelButtonText: 'Cancelar' }).then((resultado) => { if (result.isConfirmed) { $.ajax({ url: 'config.php', type: 'POST', dataType: 'json', data: { action: 'delete', id: taskId }, sucesso: function(response) { if (resposta.sucesso) { listItem.remove(); var total = parseInt($('#total-tarefas').text()) - 1; $('#total-tarefas').text(total); if (total === 0) { $('#lista-tarefas').html(''POST', tipo de dados: 'json', dados: { ação: 'excluir', id: taskId }, sucesso: função(resposta) { if (response.success) { listItem.remove(); var total = parseInt($('#total-tarefas').text()) - 1; $('#total-tarefas').text(total); if (total === 0) { $('#lista-tarefas').html(''POST', tipo de dados: 'json', dados: { ação: 'excluir', id: taskId }, sucesso: função(resposta) { if (response.success) { listItem.remove(); var total = parseInt($('#total-tarefas').text()) - 1; $('#total-tarefas').text(total); if (total === 0) { $('#lista-tarefas').html('
Nenhuma tarefa cadastrada.
'); } Swal.fire('Excluído!', resposta.message, 'sucesso'); } else { Swal.fire('Erro!', resposta.message, 'erro'); } }, erro: function() { Swal.fire('Erro de Conexão', 'Não foi possível excluir a tarefa.', 'error'); } }); } }); }); }); </script>
