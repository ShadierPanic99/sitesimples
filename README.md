<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registro de Ponto</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            padding: 0;
            background-color: #f4f4f9;
            color: #333;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        table, th, td {
            border: 1px solid #ccc;
        }
        th, td {
            padding: 10px;
            text-align: center;
        }
        th {
            background-color: #007BFF;
            color: white;
        }
        .add-row, .register-time {
            margin: 10px 0;
            padding: 10px 20px;
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
        }
        .add-row:hover, .register-time:hover {
            background-color: #218838;
        }
        #floatingClock {
            position: fixed;
            top: 10px;
            right: 10px;
            padding: 10px;
            background-color: #007BFF;
            color: white;
            font-size: 14px;
            font-weight: bold;
            border-radius: 5px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            display: flex;
            align-items: center;
            gap: 10px;
        }
    </style>
</head>
<body>
    <div id="floatingClock">
        <span id="clockDisplay"></span>
        <button class="register-time" onclick="registerTime()">Registrar</button>
    </div>

    <h1>Registro de Ponto</h1>
    <table id="timeSheet">
        <thead>
            <tr>
                <th>Data</th>
                <th>Entrada</th>
                <th>Início do Almoço</th>
                <th>Fim do Almoço</th>
                <th>Saída</th>
                <th>Horas Trabalhadas</th>
                <th>Horas Extras</th>
            </tr>
        </thead>
        <tbody id="timeSheetBody">
            <!-- Linhas serão adicionadas aqui -->
        </tbody>
    </table>
    <button class="add-row" onclick="addRow()">Adicionar Dia</button>

    <script>
        let currentStep = 0;

        function startClock() {
            const clockDisplay = document.getElementById('clockDisplay');

            function updateClock() {
                const now = new Date();
                const date = now.toLocaleDateString();
                const time = now.toLocaleTimeString();
                clockDisplay.textContent = `${date} - ${time}`;
            }

            updateClock();
            setInterval(updateClock, 1000);
        }

        function addRow() {
            const tableBody = document.getElementById('timeSheetBody');
            const row = document.createElement('tr');
            row.innerHTML = `
                <td>${new Date().toLocaleDateString()}</td>
                <td class="editable entry-time"></td>
                <td class="editable lunch-start"></td>
                <td class="editable lunch-end"></td>
                <td class="editable exit-time"></td>
                <td class="hours-worked">00:00:00</td>
                <td class="extra-hours">00:00:00</td>
            `;
            tableBody.appendChild(row);
            currentStep = 0; // Reset para nova linha

            enableEditing(row); // Permitir edição
        }

        function registerTime() {
            const tableBody = document.getElementById('timeSheetBody');
            const rows = tableBody.querySelectorAll('tr');
            if (rows.length === 0) {
                alert('Por favor, adicione uma linha primeiro.');
                return;
            }

            const lastRow = rows[rows.length - 1];
            const now = new Date();
            const timeString = `${padZero(now.getHours())}:${padZero(now.getMinutes())}:${padZero(now.getSeconds())}`;

            switch (currentStep) {
                case 0:
                    lastRow.querySelector('.entry-time').textContent = timeString;
                    currentStep++;
                    break;
                case 1:
                    lastRow.querySelector('.lunch-start').textContent = timeString;
                    currentStep++;
                    break;
                case 2:
                    lastRow.querySelector('.lunch-end').textContent = timeString;
                    currentStep++;
                    break;
                case 3:
                    lastRow.querySelector('.exit-time').textContent = timeString;
                    calculateWorkedHours(lastRow);
                    currentStep = 0; // Reset para próxima entrada
                    break;
                default:
                    alert('Todos os horários já foram registrados para esta linha.');
            }
        }

        function enableEditing(row) {
            const editableCells = row.querySelectorAll('.editable');
            editableCells.forEach(cell => {
                cell.setAttribute('contenteditable', 'true');
                cell.addEventListener('input', () => calculateWorkedHours(row));
            });
        }

        function calculateWorkedHours(row) {
            const entryTime = row.querySelector('.entry-time').textContent;
            const lunchStartTime = row.querySelector('.lunch-start').textContent;
            const lunchEndTime = row.querySelector('.lunch-end').textContent;
            const exitTime = row.querySelector('.exit-time').textContent;

            if (entryTime && lunchStartTime && lunchEndTime && exitTime) {
                const entry = parseTime(entryTime);
                const lunchStart = parseTime(lunchStartTime);
                const lunchEnd = parseTime(lunchEndTime);
                const exit = parseTime(exitTime);

                const workBeforeLunch = (lunchStart - entry) / 60000; // minutos antes do almoço
                const workAfterLunch = (exit - lunchEnd) / 60000; // minutos após o almoço
                const totalMinutes = workBeforeLunch + workAfterLunch;

                const workedHours = formatTime(totalMinutes);
                row.querySelector('.hours-worked').textContent = workedHours;

                const regularMinutes = 8 * 60; // 8 horas regulares
                if (totalMinutes > regularMinutes) {
                    const overtimeMinutes = totalMinutes - regularMinutes;
                    row.querySelector('.extra-hours').textContent = formatTime(overtimeMinutes);
                } else {
                    row.querySelector('.extra-hours').textContent = '00:00:00';
                }
            }
        }

        function parseTime(timeString) {
            const [hours, minutes] = timeString.split(':').map(Number);
            const date = new Date();
            date.setHours(hours, minutes, 0);
            return date;
        }

        function formatTime(minutes) {
            const hours = Math.floor(minutes / 60);
            const mins = Math.floor(minutes % 60);
            return `${padZero(hours)}:${padZero(mins)}:00`;
        }

        function padZero(value) {
            return value < 10 ? '0' + value : value;
        }

        // Inicia o relógio ao carregar a página
        startClock();
    </script>
</body>
</html>
