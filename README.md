<!DOCTYPE html>

<html lang="pt-BR">

<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Gerenciamento de Produtos</title>

    <script src="https://unpkg.com/@zxing/browser@latest"></script>

    <style>

        body {

            font-family: Arial, sans-serif;

            margin: 0;

            padding: 0;

            background-color: #f5f5f5;

        }

        .header {

            background-color: #f5f5f5;

            color: #000000;

            padding: 20px;

            text-align: center;

        }

        .container {

            margin: 20px;

        }

        table {

            width: 100%;

            border-collapse: collapse;

            margin-top: 20px;

        }

        th, td {

            border: 1px solid #ddd;

            padding: 10px;

            text-align: left;

        }

        th {

            background-color: #4CAF50;

            color: white;

        }

        .button {

            background-color: #4CAF50;

            color: white;

            padding: 10px 15px;

            border: none;

            border-radius: 5px;

            cursor: pointer;

            font-size: 14px;

            margin-right: 10px;

            transition: background-color 0.3s;

        }

        .button:hover {

            background-color: #45a049;

        }

        .button-danger {

            background-color: red;

            color: white;

        }

        .button-danger:hover {

            background-color: darkred;

        }

        .form-container input {

            padding: 10px;

            width: calc(100% - 20px);

            margin: 10px 0;

            border: 1px solid #ddd;

            border-radius: 5px;

        }

    </style>

</head>

<body>

    <div class="header">

        <img src="https://i.postimg.cc/QNgNgStT/Screenshot-2025-01-18-11-31-01-143-com-whatsapp.png" alt="Logo" style="max-width: 120%; height: auto;">

        <h2>Lista de Estoque e Faltas do Setor de Marcus</h2>

    </div>

    <div class="container">

        <h2>Adicionar Produtos</h2>

        <div class="form-container">

            <input type="text" id="barcodeInput" placeholder="Código de Barras" oninput="handleBarcodeInput(this.value)">

            <button class="button" onclick="startScanner()">Escanear Código de Barras</button>

            <video id="scannerPreview" style="width:100%; max-height:300px; margin-top:10px;"></video>

            <input type="text" id="productName" placeholder="Nome do Produto" required>

            <input type="number" id="productUnit" placeholder="Und" min="0" required>

            <input type="number" id="productPrice" placeholder="Preço (Opcional)" step="0.01" min="0">

            <button class="button" onclick="addProduct()">Adicionar Produto</button>

        </div>



        <h2>Lista de Produtos</h2>

        <button class="button" onclick="sortProducts('asc')">Ordenar A-Z</button>

        <button class="button" onclick="sortProducts('desc')">Ordenar Z-A</button>

        <table>

            <thead>

                <tr>

                    <th>Selecionar</th>

                    <th>Produto</th>

                    <th>Und</th>

                    <th>Preço (R$)</th>

                    <th>Ações</th>

                </tr>

            </thead>

            <tbody id="productTable"></tbody>

        </table>



        <button class="button" onclick="sendSelectedToWhatsApp()">Enviar Selecionados pelo WhatsApp</button>

    </div>



    <script>

        let editIndex = null;

        let codeReader;



        const barcodeDB = {

            "7894900011517": { name: "Arroz Tio João 1kg", price: 5.99 },

            "7891910000197": { name: "Feijão Carioca 1kg", price: 6.49 },

            "7891000055126": { name: "Açúcar União 1kg", price: 4.75 }

        };



        function handleBarcodeInput(code) {

            const product = barcodeDB[code];

            if (product) {

                document.getElementById("productName").value = product.name;

                document.getElementById("productPrice").value = product.price;

                document.getElementById("productUnit").value = 1;

            }

        }



        async function startScanner() {

            const preview = document.getElementById("scannerPreview");

            codeReader = new ZXing.BrowserBarcodeReader();



            try {

                const result = await codeReader.decodeOnceFromVideoDevice(undefined, preview);

                document.getElementById("barcodeInput").value = result.text;

                handleBarcodeInput(result.text);

                codeReader.reset();

            } catch (err) {

                console.error(err);

                alert("Falha ao ler código de barras.");

            }

        }



        function getProducts() {

            return JSON.parse(localStorage.getItem("products") || "[]");

        }



        function saveProducts(products) {

            localStorage.setItem("products", JSON.stringify(products));

        }



        function displayProducts() {

            const products = getProducts();

            const productTable = document.getElementById("productTable");

            productTable.innerHTML = "";



            products.forEach((product, index) => {

                const row = document.createElement("tr");

                row.innerHTML = `

                    <td><input type="checkbox" class="product-checkbox" data-index="${index}"></td>

                    <td>${product.name}</td>

                    <td>${product.unit || 0}</td>

                    <td>${product.price ? `R$ ${product.price.toFixed(2)}` : "-"}</td>

                    <td>

                        <button class="button button-danger" onclick="deleteProduct(${index})">Remover</button>

                    </td>

                `;

                productTable.appendChild(row);

            });

        }



        function addProduct() {

            const name = document.getElementById("productName").value.trim();

            const unit = parseInt(document.getElementById("productUnit").value) || 0;

            const price = parseFloat(document.getElementById("productPrice").value);



            if (!name) {

                alert("Por favor, insira o nome do produto.");

                return;

            }



            const products = getProducts();

            products.push({ name, unit, price: isNaN(price) ? null : price });

            saveProducts(products);

            displayProducts();

            clearForm();

        }



        function clearForm() {

            document.getElementById("barcodeInput").value = "";

            document.getElementById("productName").value = "";

            document.getElementById("productUnit").value = "";

            document.getElementById("productPrice").value = "";

        }



        function deleteProduct(index) {

            const products = getProducts();

            products.splice(index, 1);

            saveProducts(products);

            displayProducts();

        }



        function sortProducts(order) {

            const products = getProducts();



            products.sort((a, b) => {

                if (order === "asc") return a.name.localeCompare(b.name);

                return b.name.localeCompare(a.name);

            });



            saveProducts(products);

            displayProducts();

        }



        function sendSelectedToWhatsApp() {

            const selectedCheckboxes = document.querySelectorAll(".product-checkbox:checked");

            const products = getProducts();

            let selectedProducts = [];



            selectedCheckboxes.forEach(checkbox => {

                const index = parseInt(checkbox.getAttribute("data-index"));

                selectedProducts.push(products[index]);

            });



            if (selectedProducts.length === 0) {

                alert("Selecione ao menos um produto.");

                return;

            }



            let message = "Lista Das Faltas e Estoque De Marcus :\\n\\n";

            selectedProducts.forEach(product => {

                message += `Produto: ${product.name}\\nUnd: ${product.unit}\\nPreço: R$ ${product.price ? product.price.toFixed(2) : "-"}\\n\\n`;

            });



            const url = `https://wa.me/5584999805340?text=${encodeURIComponent(message)}`;

            window.open(url, "_blank");

        }



        window.onload = displayProducts;

    </script>

</body>

</html>

'''



