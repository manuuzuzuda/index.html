# index.html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TaskFlow - Gestão de Tarefas Acessível</title>
    <style>
        /* Ajuste de Contraste e Foco Visual */
        body {
            font-family: Arial, sans-serif;
            background-color: #f9f9f9; /* Fundo ligeiramente mais claro para contraste */
            color: #1a1a1a; /* Texto mais escuro (Contraste aumentado de #333 para #1a1a1a) */
            margin: 0;
            padding: 20px;
        }

        h1 {
            text-align: center;
            color: #0b401c; /* Verde escuro acessível para o título */
        }

        .task-input {
            display: flex;
            justify-content: center;
            margin-bottom: 20px;
        }

        .task-input input {
            padding: 10px;
            font-size: 16px;
            border: 2px solid #666;
            border-radius: 4px 0 0 4px;
        }

        /* Botão Adicionar: Verde alterado para atender WCAG AAA (> 7:1) */
        .task-input button {
            padding: 10px;
            background-color: #19692c; 
            color: white;
            border: none;
            cursor: pointer;
            font-weight: bold;
            border-radius: 0 4px 4px 0;
        }

        .task-list {
            list-style: none;
            padding: 0;
            margin: 0;
            max-width: 600px;
            margin: 0 auto;
        }

        .task-list li {
            background-color: white;
            margin: 10px 0;
            padding: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        /* Estilo customizado de tarefa concluída (Mantendo o contraste do texto) */
        .task-list li.completed {
            background-color: #e2f0d9;
            border-color: #b3d89c;
        }
        
        .task-list li.completed span {
            text-decoration: line-through;
            color: #4a4a4a; /* Cinza escuro para manter leitura com o risco */
        }

        /* Botões de Ação da Lista */
        .btn-complete {
            background-color: #0b5ed7; /* Azul perceptível */
            color: white;
            border: none;
            cursor: pointer;
            padding: 8px 12px;
            border-radius: 4px;
            margin-right: 5px;
        }

        .btn-delete {
            background-color: #bb2d3b; /* Vermelho escuro para contraste do texto 'Excluir' */
            color: white;
            border: none;
            cursor: pointer;
            padding: 8px 12px;
            border-radius: 4px;
        }

        /* Indicador de foco visível - CRUCIAL PARA NAVEGAÇÃO POR TECLADO */
        *:focus {
            outline: 3px solid #ffc107 !important;
            outline-offset: 2px;
        }

        /* Classe utilitária para leitores de tela (invisível visualmente) */
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</head>
<body>

    <main>
        <h1>TaskFlow</h1>
        
        <p class="text-center" style="text-align:center; font-size: 0.9rem; color: #555;">
            Dica: Use o atalho <kbd>Ctrl + N</kbd> para ir direto para o campo de digitação.
        </p>
    </main>

    <section class="task-input">
        <input type="text" id="new-task" placeholder="Digite sua tarefa" aria-label="Nova tarefa">
        <button id="add-task">Adicionar Tarefa</button>
    </section>

    <div id="feedback" class="sr-only" aria-live="polite"></div>

    <section>
        <ul class="task-list" id="task-list" aria-label="Lista de tarefas pendentes e concluídas">
            </ul>
    </section>

    <script>
        const taskInput = document.getElementById('new-task');
        const addTaskButton = document.getElementById('add-task');
        const taskList = document.getElementById('task-list');
        const feedback = document.getElementById('feedback');

        // Função para atualizar o leitor de tela de forma limpa
        function announceToScreenReader(message) {
            feedback.textContent = ''; // Limpa o estado anterior
            setTimeout(() => {
                feedback.textContent = message;
            }, 100);
        }

        function addTask(taskText) {
            const taskItem = document.createElement('li');
            
            // Geramos IDs únicos para vincular o texto da tarefa aos botões usando aria-describedby
            const taskId = 'task-' + Date.now();
            
            taskItem.innerHTML = `
                <span id="${taskId}">${taskText}</span>
                <div>
                    <button class="btn-complete" onclick="completeTask(this)" aria-describedby="${taskId}">Concluir</button>
                    <button class="btn-delete" onclick="removeTask(this)" aria-describedby="${taskId}">Excluir</button>
                </div>
            `;
            taskList.appendChild(taskItem);
            announceToScreenReader(`Tarefa "${taskText}" adicionada com sucesso.`);
        }

        function completeTask(button) {
            const taskItem = button.parentElement.parentElement;
            const taskText = taskItem.querySelector('span').textContent;
            
            taskItem.classList.toggle('completed');
            
            const isCompleted = taskItem.classList.contains('completed');
            if (isCompleted) {
                button.setAttribute('aria-pressed', 'true');
                announceToScreenReader(`Tarefa "${taskText}" marcada como concluída.`);
            } else {
                button.setAttribute('aria-pressed', 'false');
                announceToScreenReader(`Tarefa "${taskText}" marcada como pendente.`);
            }
        }

        function removeTask(button) {
            const taskItem = button.parentElement.parentElement;
            const taskText = taskItem.querySelector('span').textContent;
            
            taskList.removeChild(taskItem);
            announceToScreenReader(`Tarefa "${taskText}" excluída.`);
            taskInput.focus(); // Devolve o foco para o input após a exclusão
        }

        // Evento de clique
        addTaskButton.addEventListener('click', () => {
            const taskText = taskInput.value.trim();
            if (taskText) {
                addTask(taskText);
                taskInput.value = '';
                taskInput.focus();
            }
        });

        // Evento de tecla enter no input
        taskInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                addTaskButton.click();
            }
        });

        // Atalho de Teclado global: Ctrl + N foca no input
        document.addEventListener('keydown', function(event) {
            if (event.ctrlKey && event.key.toLowerCase() === 'n') {
                event.preventDefault(); // Evita comportamentos padrões do navegador
                taskInput.focus();
            }
        });
    </script>
</body>
</html>
