
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Barbearia System</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f5f5f5;
        }

        header {
            background-color: #009688;
            color: white;
            padding: 1rem ;
            text-align: center;
        }

        main {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 20px;
        }

        label {
            font-weight: bold;
        }

        input,
        select,
        button {
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin-top: 5px;
        }

        button {
            background-color: #009688;
            color: white;
            border: none;
            cursor: pointer;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
            font-size: 14px;
        }

        th,
        td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }

        h3 {
            margin-top: 0;
        }

        #result {
            margin-top: 20px;
        }

        .delete-button {
            background-color: #ff4500;
            color: white;
            border: none;
            cursor: pointer;
        }

        #customCostInput {
            margin-top: 10px;
        }

        @media only screen and (max-width: 600px) {
            body {
                padding: 0.5em;
            }

            main {
                margin: 10px auto;
                padding: 10px;
            }

            form {
                margin-bottom: 10px;
            }
        }
    </style>
</head>

<body>
    <header>
        <h1>Barbearia Italo</h1>
    </header>
    <main>
        <form id="barbersForm">
            <label for="barberName">Nome do Funcionario:</label>
            <input type="text" id="barberName" required>
            <table id="barbersTable">
                <thead>
                    <tr>
                        <th>Funcionarios cadastrados: </th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
            <button type="button" onclick="registerBarber()">Registrar Funcionario</button>
        </form>
        <form id="servicesForm">
            <label for="barberSelect">Funcionario:</label>
            <select id="barberSelect" required></select>
            <label for="serviceType">Tipo de Servico:</label>
            <select id="serviceType" required>
                <option value="corte">Corte - R$20,00</option>
                <option value="corte infantil">Corte Infantil - R$22,00</option>
                <option value="corte maquina">Corte Maquina - R$16,00</option>
                <option value="barba">Barba - R$18,00</option>
                <option value="sobrancelha">Sobrancelha - R$8,00</option>
                <option value="pezinho">Pezinho - R$10,00</option>
                <option value="corte-sobrancelha">Corte + Sobrancelha - R$26,00</option>
                <option value="corte-barba">Corte + Barba - R$36,00</option>
                <option value="corte-barba-sobrancelha">Corte + Barba + Sobrancelha - R$42,00</option>
            </select>
            <div id="customCostInput">
                <label for="customCost">Valor Alternativo:</label>
                <input type="number" id="customCost" step="0.02" placeholder="Insira o valor alternativo">
            </div>
            <label for="serviceDate">Data do Servico:</label>
            <input type="date" id="serviceDate" required>
            <button type="button" onclick="recordService()">Registrar Servico</button>
        </form>
        <table id="servicesTable">
            <thead>
                <tr>
                    <th>Funcionario</th>
                    <th>Servico</th>
                    <th>Valor</th>
                    <th>Data</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
        <button type="button" onclick="calculatePayments()">Calcular Pagamentos</button>
        <div id="result"></div>
    </main>
    <script>
        let barbers = [];
        let services = [];

        window.addEventListener("load", function () {
            loadBarbers();
            loadServices();
        });

        function registerBarber() {
            const barberName = document.getElementById("barberName").value;
            if (!barbers.includes(barberName)) {
                barbers.push(barberName);
                updateBarberSelect();
                updateBarbersTable();
                saveDataToLocalStorage();
            } else {
                alert("Este funcionario ja esta registrado.");
            }
            document.getElementById("barberName").value = "";
        }

        function updateBarberSelect() {
            const select = document.getElementById("barberSelect");
            select.innerHTML = "";
            barbers.forEach(barber => {
                const option = document.createElement("option");
                option.value = barber;
                option.textContent = barber;
                select.appendChild(option);
            });
        }

        function updateBarbersTable() {
            const tableBody = document.querySelector("#barbersTable tbody");
            tableBody.innerHTML = "";
            barbers.forEach(barber => {
                const row = tableBody.insertRow();
                const cell1 = row.insertCell(0);
                const cell2 = row.insertCell(1);
                cell1.textContent = barber;
                const deleteButton = document.createElement("button");
                deleteButton.textContent = "Excluir";
                deleteButton.className = "delete-button";
                deleteButton.onclick = () => deleteBarber(barber);
                cell2.appendChild(deleteButton);
            });
        }

        function deleteBarber(barber) {
            const index = barbers.indexOf(barber);
            if (index !== -1) {
                barbers.splice(index, 1);
                updateBarberSelect();
                updateBarbersTable();
                saveDataToLocalStorage();
            }
        }

        function loadBarbers() {
            const storedBarbers = localStorage.getItem("barbers");
            if (storedBarbers) {
                barbers = JSON.parse(storedBarbers);
                updateBarberSelect();
                updateBarbersTable();
            }
        }

        function recordService() {
            const barberName = document.getElementById("barberSelect").value;
            const serviceType = document.getElementById("serviceType").value;
            const customCostInput = document.getElementById("customCost");
            let serviceCost;
            if (customCostInput.value.trim() !== "") {
                serviceCost = parseFloat(customCostInput.value);
            } else {
                serviceCost = getServiceCost(serviceType);
            }
            const serviceDate = document.getElementById("serviceDate").value;
            services.push({ barberName, serviceType, serviceCost, serviceDate });
            updateServicesTable();
            saveDataToLocalStorage();
        }

        function updateServicesTable() {
            const tableBody = document.querySelector("#servicesTable tbody");
            tableBody.innerHTML = "";
            services.forEach((service, index) => {
                const row = tableBody.insertRow();
                const cell1 = row.insertCell(0);
                const cell2 = row.insertCell(1);
                const cell3 = row.insertCell(2);
                const cell4 = row.insertCell(3);
                const cell5 = row.insertCell(4);
                cell1.textContent = service.barberName;
                cell2.textContent = service.serviceType;
                cell3.textContent = `R$ ${service.serviceCost.toFixed(2)}`;
                cell4.textContent = formatDate(service.serviceDate);
                const deleteButton = document.createElement("button");
                deleteButton.textContent = "Excluir";
                deleteButton.className = "delete-button";
                deleteButton.onclick = () => deleteService(index);
                cell5.appendChild(deleteButton);
            });
        }

        function formatDate(dateString) {
            const date = new Date(dateString + "T00:00:00");
            const options = { day: '2-digit', month: '2-digit', year: 'numeric' };
            const formattedDate = date.toLocaleDateString('pt-BR', options);
            return formattedDate.split('/').join('/');
        }

        function deleteService(index) {
            services.splice(index, 1);
            updateServicesTable();
            saveDataToLocalStorage();
        }

        function calculatePayments() {
            const payments = {};
            const totalAmounts = {};
            const dailyTotals = {};
            services.forEach(service => {
                const grossPayment = service.serviceCost;
                if (!dailyTotals[service.serviceDate]) {
                    dailyTotals[service.serviceDate] = {};
                }
                if (!dailyTotals[service.serviceDate][service.barberName]) {
                    dailyTotals[service.serviceDate][service.barberName] = grossPayment;
                } else {
                    dailyTotals[service.serviceDate][service.barberName] += grossPayment;
                }
                const netPayment = 0.4 * service.serviceCost;
                if (!payments[service.barberName]) {
                    payments[service.barberName] = netPayment;
                } else {
                    payments[service.barberName] += netPayment;
                }
                if (!totalAmounts[service.serviceDate]) {
                    totalAmounts[service.serviceDate] = grossPayment;
                } else {
                    totalAmounts[service.serviceDate] += grossPayment;
                }
            });
            const resultElement = document.getElementById("result");
            resultElement.innerHTML = "";
            for (const date in dailyTotals) {
                resultElement.innerHTML += `<h3>${getDayOfWeek(date)} - ${formatDate(date)}</h3>`;
                for (const barber in dailyTotals[date]) {
                    const grossAmount = dailyTotals[date][barber];
                    const netAmount = 0.4 * grossAmount;
                    resultElement.innerHTML += `<p>${barber}: R$ ${netAmount.toFixed(2)}</p>`;
                }
                const dailyTotal = Object.values(dailyTotals[date]).reduce((sum, value) => sum + value, 0);
                resultElement.innerHTML += `<p>Total diario: R$ ${dailyTotal.toFixed(2)}</p>`;
                resultElement.innerHTML += `<hr>`;
            }
            for (const barber in payments) {
                resultElement.innerHTML += `<p>${barber}: R$ ${payments[barber].toFixed(2)}</p>`;
            }
            const totalWeekly = Object.values(totalAmounts).reduce((sum, value) => sum + value, 0);
            resultElement.innerHTML += `<p>Total semanal: R$ ${totalWeekly.toFixed(2)}</p>`;
        }

        function calculateTotalAmount(totalAmounts, currentDate) {
            return Object.keys(totalAmounts)
                .filter(date => new Date(date + "T00:00:00") <= new Date(currentDate + "T00:00:00"))
                .reduce((sum, date) => sum + totalAmounts[date], 0);
        }

        function getDayOfWeek(date) {
            const daysOfWeek = ['Segunda-feira', 'Terca-feira', 'Quarta-feira', 'Quinta-feira', 'Sexta-feira', 'Sabado', 'Domingo'];
            const dayIndex = new Date(date).getDay();
            return daysOfWeek[dayIndex];
        }

        function saveDataToLocalStorage() {
            localStorage.setItem("barbers", JSON.stringify(barbers));
            localStorage.setItem("services", JSON.stringify(services));
        }

        function loadServices() {
            const storedServices = localStorage.getItem("services");
            if (storedServices) {
                services = JSON.parse(storedServices);
                updateServicesTable();
            }
        }

        function getServiceCost(serviceType) {
            switch (serviceType) {
                case "corte":
                    return 20;
                case "corte infantil":
                    return 22;
                case "corte maquina":
                    return 15;
                case "barba":
                    return 18;
                case "sobrancelha":
                    return 8;
                case "pezinho":
                    return 10;
                case "corte-sobrancelha":
                    return 26;
                case "corte-barba":
                    return 36;
                case "corte-barba-sobrancelha":
                    return 42;
                default:
                    return 0;
            }
        }
    </script>
</body>

</html>
