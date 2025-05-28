<?php
$conexao = new PDO(
    "mysql:dbname=biblioteca;host=localhost", "root", "ceub123456"
);
$conexao->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$mensagem = '';

// Handle DELETE
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'delete') {
    $id = $_POST['delete_id'];
    $stmt = $conexao->prepare("DELETE FROM livros WHERE id = :id");
    $stmt->bindParam(':id', $id, PDO::PARAM_INT);
    if ($stmt->execute()) {
        $mensagem = "Livro deletado com sucesso!";
    } else {
        $mensagem = "Erro ao deletar livro.";
    }
}

// Handle INSERT
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'add') {
    $titulo = $_POST['titulo'];
    $autor_id = $_POST['autor_id'];
    $ano = $_POST['ano_publicacao'];
    $descricao = $_POST['descricao'];
    $disponibilidade = $_POST['disponibilidade'];

    $stmt = $conexao->prepare("
        INSERT INTO livros (titulo, autor_id, ano_publicacao, descricao, disponibilidade) 
        VALUES (:titulo, :autor_id, :ano, :descricao, :disponibilidade)
    ");
    $stmt->bindParam(':titulo', $titulo);
    $stmt->bindParam(':autor_id', $autor_id);
    $stmt->bindParam(':ano', $ano);
    $stmt->bindParam(':descricao', $descricao);
    $stmt->bindParam(':disponibilidade', $disponibilidade);

    if ($stmt->execute()) {
        $mensagem = "Livro adicionado com sucesso!";
    } else {
        $mensagem = "Erro ao adicionar livro.";
    }
}

// Buscar autores para o select do formulário
$autoresQuery = $conexao->query("SELECT * FROM autores");
$autores = $autoresQuery->fetchAll();

// Buscar livros com nome do autor
$sql = $conexao->query("
    SELECT livros.*, autores.nome AS nome_autor 
    FROM livros 
    JOIN autores ON livros.autor_id = autores.id
");
$livros = $sql->fetchAll();
?>
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Biblioteca</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container mt-5">
        <h1 class="mb-4 text-center">Biblioteca - Inserção e Lista de Livros</h1>

        <!-- Mensagem de feedback -->
        <?php if ($mensagem) { ?>
            <div class="alert alert-info"><?= $mensagem ?></div>
        <?php } ?>

        <!-- Formulário de Inserção -->
        <div class="card mb-4">
            <div class="card-header bg-primary text-white">
                Adicionar Novo Livro
            </div>
            <div class="card-body">
                <form method="POST">
                    <input type="hidden" name="action" value="add">
                    <div class="row mb-3">
                        <div class="col">
                            <label class="form-label">Título</label>
                            <input type="text" name="titulo" class="form-control" required>
                        </div>
                        <div class="col">
                            <label class="form-label">Autor</label>
                            <select name="autor_id" class="form-select" required>
                                <option value="">Selecione</option>
                                <?php foreach ($autores as $autor) { ?>
                                    <option value="<?= $autor['id'] ?>"><?= htmlspecialchars($autor['nome']) ?></option>
                                <?php } ?>
                            </select>
                        </div>
                        <div class="col">
                            <label class="form-label">Ano</label>
                            <input type="number" name="ano_publicacao" class="form-control" required>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Descrição</label>
                        <input type="text" name="descricao" class="form-control" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Disponibilidade</label>
                        <select name="disponibilidade" class="form-select" required>
                            <option value="1">Disponível</option>
                            <option value="0">Indisponível</option>
                        </select>
                    </div>
                    <button type="submit" class="btn btn-success">Adicionar Livro</button>
                </form>
            </div>
        </div>

        <!-- Tabela de Livros -->
        <div class="table-responsive">
            <table class="table table-striped table-hover table-bordered align-middle">
                <thead class="table-dark">
                    <tr>
                        <th>Título</th>
                        <th>Autor</th>
                        <th>Ano</th>
                        <th>Descrição</th>
                        <th>Disponibilidade</th>
                        <th>Ações</th>
                    </tr>
                </thead>
                <tbody>
                    <?php foreach($livros as $livro){ ?>
                    <tr>
                        <td><?= htmlspecialchars($livro['titulo']) ?></td>
                        <td><?= htmlspecialchars($livro['nome_autor']) ?></td>
                        <td><?= htmlspecialchars($livro['ano_publicacao']) ?></td>
                        <td><?= htmlspecialchars($livro['descricao']) ?></td>
                        <td>
                            <?php if ($livro['disponibilidade']) { ?>
                                <span class="badge bg-success">Disponível</span>
                            <?php } else { ?>
                                <span class="badge bg-danger">Indisponível</span>
                            <?php } ?>
                        </td>
                        <td>
                            <form method="POST" style="display:inline;" onsubmit="return confirm('Tem certeza que deseja deletar este livro?');">
                                <input type="hidden" name="action" value="delete">
                                <input type="hidden" name="delete_id" value="<?= $livro['id'] ?>">
                                <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                            </form>
                        </td>
                    </tr>
                    <?php } ?>
                </tbody>
            </table>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
