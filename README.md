<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minhas Dívidas</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f9f9f9;
            margin: 0;
            padding: 0;
            text-align: center;
        }
        header {
            background-color: #2c3e50;
            color: white;
            padding: 20px;
            font-size: 24px;
            font-weight: bold;
        }
        main {
            padding: 20px;
        }
        form {
            background-color: white;
            max-width: 500px;
            margin: 20px auto;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }
        label {
            display: block;
            margin: 10px 0 5px;
            font-weight: bold;
        }
        input[type="text"],
        input[type="number"],
        input[type="date"] {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            background-color: #2c3e50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #273c4f;
        }
        .debt-list {
            margin-top: 20px;
            text-align: left;
            max-width: 500px;
            margin: 0 auto;
        }
        .debt-item {
            background-color: white;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 4px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .debt-item.overdue {
            background-color: #f8d7da;
            color: #721c24;
        }
        .delete-btn, .edit-btn, .pay-btn {
            margin-left: 5px;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        .delete-btn {
            background-color: #e74c3c;
            color: white;
        }
        .delete-btn:hover {
            background-color: #c0392b;
        }
        .edit-btn {
            background-color: #ffc107;
            color: black;
        }
        .edit-btn:hover {
            background-color: #e0a800;
        }
        .pay-btn {
            background-color: #28a745;
            color: white;
        }
        .pay-btn:hover {
            background-color: #218838;
        }
        .total-debt {
            font-size: 18px;
            font-weight: bold;
            margin-top: 20px;
        }
        .notification {
            background-color: #f8d7da;
            color: #721c24;
            padding: 10px;
            border-radius: 4px;
            margin: 20px auto;
            max-width: 500px;
        }
        .qr-code {
            margin-top: 20px;
        }
        .qr-code img {
            max-width: 150px;
        }
    </style>
</head>
<body>

    <header>
        Minhas Dívidas
    </header>

    <main>
        <h2>Cadastrar Nova Conta</h2>
        <form id="debtForm">
            <label for="debtName">Nome da Conta:</label>
            <input type="text" id="debtName" placeholder="Ex: Energia Elétrica" required>

            <label for="debtAmount">Valor (R$):</label>
            <input type="number" id="debtAmount" step="0.01" placeholder="Ex: 150.00" required>

            <label for="dueDate">Data de Vencimento:</label>
            <input type="date" id="dueDate" required>

            <button type="submit">Cadastrar Conta</button>
        </form>

        <div class="debt-list" id="debtList">
            <h3>Contas Cadastradas</h3>
        </div>

        <div class="total-debt" id="totalDebt"></div>

        <div class="notification" id="notifications"></div>

        <button id="exportCSV">Exportar para CSV</button>

        <!-- QR Code -->
        <div class="qr-code">
            <h3>Acesse pelo Celular</h3>
            <img id="qrCodeImage" src="" alt="QR Code">
        </div>

    </main>

    <script>
        // Função para carregar as dívidas salvas no localStorage
        function loadDebts() {
            const savedDebts = localStorage.getItem('debts');
            if (savedDebts) {
                return JSON.parse(savedDebts);
            }
            return [];
        }

        // Função para salvar as dívidas no localStorage
        function saveDebts(debts) {
            localStorage.setItem('debts', JSON.stringify(debts));
        }

        // Função para renderizar a lista de dívidas
        function renderDebtList() {
            const debtList = document.getElementById('debtList');
            debtList.innerHTML = '<h3>Contas Cadastradas</h3>';

            const debts = loadDebts();
            let totalDebt = 0;

            debts.forEach((debt, index) => {
                const today = new Date();
                const dueDate = new Date(debt.dueDate);
                const isOverdue = dueDate < today && !debt.paid; // Verifica se a conta está atrasada e não foi paga

                const debtItem = document.createElement('div');
                debtItem.classList.add('debt-item');
                if (isOverdue) {
                    debtItem.classList.add('overdue'); // Adiciona classe para contas atrasadas
                }

                debtItem.innerHTML = `
                    <div>
                        <strong>${debt.name}</strong><br>
                        Valor: R$ ${parseFloat(debt.amount).toFixed(2)}<br>
                        Vencimento: ${debt.dueDate}${debt.paid ? ' - PAGO' : ''}
                    </div>
                    <div>
                        ${!debt.paid ? `<button class="pay-btn" data-index="${index}">Pagar</button>` : ''}
                        <button class="edit-btn" data-index="${index}">Editar</button>
                        <button class="delete-btn" data-index="${index}">Excluir</button>
                    </div>
                `;
                debtList.appendChild(debtItem);

                if (!debt.paid) {
                    totalDebt += parseFloat(debt.amount);
                }
            });

            // Exibe o valor total das dívidas
            document.getElementById('totalDebt').textContent = `Total a Pagar: R$ ${totalDebt.toFixed(2)}`;

            // Adiciona evento de exclusão aos botões
            document.querySelectorAll('.delete-btn').forEach(button => {
                button.addEventListener('click', function() {
                    const index = this.dataset.index;
                    deleteDebt(index);
                });
            });

            // Adiciona evento para marcar como paga
            document.querySelectorAll('.pay-btn').forEach(button => {
                button.addEventListener('click', function() {
                    const index = this.dataset.index;
                    markAsPaid(index);
                });
            });

            // Adiciona evento para editar uma conta
            document.querySelectorAll('.edit-btn').forEach(button => {
                button.addEventListener('click', function() {
                    const index = this.dataset.index;
                    editDebt(index);
                });
            });

            // Verifica notificações de vencimento
            checkNotifications(debts);
        }

        // Função para adicionar ou editar uma dívida
        function handleDebtFormSubmit(event, indexToEdit = null) {
            event.preventDefault();

            const name = document.getElementById('debtName').value;
            const amount = document.getElementById('debtAmount').value;
            const dueDate = document.getElementById('dueDate').value;

            const newDebt = { name, amount, dueDate, paid: false };

            const debts = loadDebts();

            if (indexToEdit !== null) {
                // Edita a dívida existente
                debts[indexToEdit] = newDebt;
            } else {
                // Adiciona uma nova dívida
                debts.push(newDebt);
            }

            saveDebts(debts);
            renderDebtList();

            // Limpa o formulário
            document.getElementById('debtName').value = '';
            document.getElementById('debtAmount').value = '';
            document.getElementById('dueDate').value = '';

            // Remove o índice de edição após salvar
            document.getElementById('debtForm').dataset.editIndex = null;
        }

        // Função para excluir uma dívida
        function deleteDebt(index) {
            const debts = loadDebts();
            debts.splice(index, 1);
            saveDebts(debts);
            renderDebtList();
        }

        // Função para marcar uma dívida como paga
        function markAsPaid(index) {
            const debts = loadDebts();
            debts[index].paid = true;
            saveDebts(debts);
            renderDebtList();
        }

        // Função para editar uma dívida
        function editDebt(index) {
            const debts = loadDebts();
            const debtToEdit = debts[index];

            // Preenche o formulário com os dados da dívida
            document.getElementById('debtName').value = debtToEdit.name;
            document.getElementById('debtAmount').value = debtToEdit.amount;
            document.getElementById('dueDate').value = debtToEdit.dueDate;

            // Define o índice de edição no formulário
            document.getElementById('debtForm').dataset.editIndex = index;

            // Altera o texto do botão de envio do formulário
            document.querySelector('#debtForm button').textContent = 'Salvar Alterações';
        }

        // Função para verificar notificações de vencimento
        function checkNotifications(debts) {
            const today = new Date();
            const notifications = [];

            debts.forEach(debt => {
                if (!debt.paid) {
                    const dueDate = new Date(debt.dueDate);
                    const timeDiff = dueDate - today;
                    const daysDiff = Math.ceil(timeDiff / (1000 * 60 * 60 * 24));

                    if (daysDiff >= 0 && daysDiff <= 7) {
                        notifications.push(`Atenção: A conta "${debt.name}" vence em ${daysDiff} dia(s)!`);
                    } else if (daysDiff < 0) {
                        notifications.push(`A conta "${debt.name}" está atrasada!`);
                    }
                }
            });

            const notificationDiv = document.getElementById('notifications');
            if (notifications.length > 0) {
                notificationDiv.innerHTML = notifications.join('<br>');
            } else {
                notificationDiv.innerHTML = '';
            }
        }

        // Função para exportar para CSV
        document.getElementById('exportCSV').addEventListener('click', function() {
            const debts = loadDebts();
            let csvContent = "data:text/csv;charset=utf-8,";

            // Cabeçalho do CSV
            csvContent += "Nome,Valor,Vencimento,Pago\n";

            // Dados das dívidas
            debts.forEach(debt => {
                csvContent += `${debt.name},${debt.amount},${debt.dueDate},${debt.paid ? 'Sim' : 'Não'}\n`;
            });

            // Cria um link para download
            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "minhas_dividas.csv");
            document.body.appendChild(link);
            link.click();
        });

        // Função para gerar QR Code
        function generateQRCode() {
            const url = window.location.href; // URL atual da página
            const qrCodeUrl = `https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=${encodeURIComponent(url)}`;
            document.getElementById('qrCodeImage').src = qrCodeUrl;
        }

        // Evento de envio do formulário
        document.getElementById('debtForm').addEventListener('submit', function(event) {
            const editIndex = this.dataset.editIndex;
            handleDebtFormSubmit(event, editIndex !== null ? parseInt(editIndex) : null);
        });

        // Carrega as dívidas ao iniciar a página
        renderDebtList();

        // Gera o QR Code ao carregar a página
        generateQRCode();
    </script>

</body>
</html>
