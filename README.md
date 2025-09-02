<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Assistente de Compras Inteligente</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Estilo para o placeholder do input de arquivo */
        input[type="file"] {
            display: none;
        }
        /* Efeito de carregamento */
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #09f;
            animation: spin 1s ease infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        /* Animação de entrada suave */
        .fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div class="container mx-auto p-4 max-w-2xl">
        <header class="text-center mb-6">
            <h1 class="text-3xl font-bold text-gray-900">🛒 Assistente de Compras</h1>
            <p class="text-gray-600">Fotografe seus produtos e controle seus gastos em tempo real.</p>
        </header>

        <!-- Seção de Orçamento Inicial -->
        <div id="budget-section" class="bg-white p-6 rounded-2xl shadow-md fade-in">
            <h2 class="text-xl font-semibold mb-4">Qual o seu orçamento?</h2>
            <div class="flex items-center space-x-2">
                <span class="text-gray-600 text-lg">R$</span>
                <input type="number" id="budget-input" placeholder="Ex: 350.50" class="flex-grow p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
            </div>
            <button id="set-budget-btn" class="mt-4 w-full bg-blue-600 text-white font-semibold py-3 rounded-lg hover:bg-blue-700 transition-transform transform hover:scale-102 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                Iniciar Compras
            </button>
        </div>

        <!-- Seção Principal do App (inicialmente oculta) -->
        <main id="main-app" class="hidden">
            <!-- Painel de Status -->
            <div id="status-panel" class="bg-white p-5 rounded-2xl shadow-lg mb-6 sticky top-4 z-10 fade-in">
                <div class="grid grid-cols-3 gap-4 text-center">
                    <div>
                        <p class="text-sm text-gray-500">Orçamento</p>
                        <p id="budget-display" class="text-lg font-bold text-blue-600">R$ 0,00</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Total Gasto</p>
                        <p id="total-spent" class="text-lg font-bold text-gray-800">R$ 0,00</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Saldo</p>
                        <p id="remaining-balance" class="text-lg font-bold text-green-600">R$ 0,00</p>
                    </div>
                </div>
            </div>

            <!-- Lista de Itens -->
            <div id="item-list" class="space-y-3">
                <!-- Itens serão adicionados aqui -->
            </div>
            
            <div id="empty-state" class="text-center py-10 px-4">
                <svg xmlns="http://www.w3.org/2000/svg" class="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
                </svg>
                <h3 class="mt-2 text-lg font-medium text-gray-900">Seu carrinho está vazio</h3>
                <p class="mt-1 text-sm text-gray-500">Clique no botão abaixo para adicionar o primeiro item.</p>
            </div>

            <!-- Botão Flutuante para Adicionar Item -->
            <div class="fixed bottom-6 right-6">
                 <button id="add-item-btn" class="bg-blue-600 text-white rounded-full p-4 shadow-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transform transition-transform hover:scale-110">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                    </svg>
                    <span class="sr-only">Adicionar Item</span>
                </button>
            </div>
            <input type="file" id="image-input" accept="image/*" capture="environment">
        </main>

        <!-- Modal para Adicionar Item -->
        <div id="add-item-modal" class="hidden fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50 p-4">
            <div class="bg-white rounded-2xl shadow-xl w-full max-w-md p-6 transform transition-all fade-in">
                <h2 class="text-xl font-bold mb-4">Adicionar Novo Item</h2>
                <div class="mb-4">
                    <img id="item-preview" src="" class="w-full h-48 object-contain rounded-lg bg-gray-200 hidden">
                    <div id="ai-loader" class="flex flex-col items-center justify-center h-48 bg-gray-200 rounded-lg">
                        <div class="spinner"></div>
                        <p class="mt-3 text-gray-600">Analisando imagem...</p>
                    </div>
                </div>

                <div class="space-y-4">
                    <div>
                        <label for="product-name-input" class="block text-sm font-medium text-gray-700">Nome do Produto (sugerido pela IA)</label>
                        <input type="text" id="product-name-input" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                           <label for="quantity-input" class="block text-sm font-medium text-gray-700">Quantidade</label>
                           <input type="number" id="quantity-input" value="1" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                        </div>
                        <div>
                           <label for="price-input" class="block text-sm font-medium text-gray-700">Preço (unid.)</label>
                           <input type="number" id="price-input" placeholder="R$ 0,00" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                        </div>
                    </div>
                </div>
                
                <div class="mt-6 flex justify-end space-x-3">
                    <button id="cancel-add-item-btn" class="bg-gray-200 text-gray-700 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition">Cancelar</button>
                    <button id="confirm-add-item-btn" class="bg-blue-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-700 transition" disabled>Adicionar</button>
                </div>
            </div>
        </div>

    </div>

    <script>
        // Variáveis de estado do aplicativo
        let budget = 0;
        let items = [];

        // Elementos do DOM
        const budgetSection = document.getElementById('budget-section');
        const mainApp = document.getElementById('main-app');
        const budgetInput = document.getElementById('budget-input');
        const setBudgetBtn = document.getElementById('set-budget-btn');
        
        const budgetDisplay = document.getElementById('budget-display');
        const totalSpentDisplay = document.getElementById('total-spent');
        const remainingBalanceDisplay = document.getElementById('remaining-balance');
        
        const itemList = document.getElementById('item-list');
        const emptyState = document.getElementById('empty-state');

        const addItemBtn = document.getElementById('add-item-btn');
        const imageInput = document.getElementById('image-input');

        const addItemModal = document.getElementById('add-item-modal');
        const itemPreview = document.getElementById('item-preview');
        const aiLoader = document.getElementById('ai-loader');
        const productNameInput = document.getElementById('product-name-input');
        const quantityInput = document.getElementById('quantity-input');
        const priceInput = document.getElementById('price-input');
        const confirmAddItemBtn = document.getElementById('confirm-add-item-btn');
        const cancelAddItemBtn = document.getElementById('cancel-add-item-btn');
        
        // --- FUNÇÕES AUXILIARES ---

        /**
         * Formata um número para o formato de moeda BRL.
         * @param {number} value - O número a ser formatado.
         * @returns {string} O valor formatado como moeda.
         */
        const formatCurrency = (value) => {
            return new Intl.NumberFormat('pt-BR', {
                style: 'currency',
                currency: 'BRL'
            }).format(value);
        };

        /**
         * Converte um arquivo de imagem para uma string base64.
         * @param {File} file - O arquivo de imagem.
         * @returns {Promise<string>} A string base64 da imagem.
         */
        const imageToBase64 = (file) => {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]);
                reader.onerror = (error) => reject(error);
            });
        };

        // --- FUNÇÕES DA API GEMINI ---

        /**
         * Chama a API do Gemini para identificar o produto na imagem.
         * @param {string} base64Image - A imagem em formato base64.
         */
        async function callGeminiAPI(base64Image) {
            const apiKey = ""; // A chave será injetada pelo ambiente de execução
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{
                    parts: [
                        { text: "Descreva o produto principal nesta imagem em poucas palavras. Seja direto. Por exemplo: 'Leite Integral 1L' ou 'Banana prata'." },
                        { inlineData: { mimeType: "image/jpeg", data: base64Image } }
                    ]
                }],
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                if (!response.ok) {
                    throw new Error(`Erro na API: ${response.statusText}`);
                }
                const result = await response.json();
                const text = result.candidates?.[0]?.content?.parts?.[0]?.text;

                if (text) {
                    productNameInput.value = text.trim().replace(/\n/g, ' ');
                } else {
                    productNameInput.value = 'Produto não identificado';
                }
            } catch (error) {
                console.error("Falha ao chamar a API Gemini:", error);
                productNameInput.value = 'Erro ao identificar';
            } finally {
                aiLoader.classList.add('hidden');
                confirmAddItemBtn.disabled = false;
                productNameInput.focus();
            }
        }

        // --- FUNÇÕES DE LÓGICA DO APP ---

        /**
         * Atualiza todo o painel de status e a lista de itens.
         */
        const updateUI = () => {
            const totalSpent = items.reduce((sum, item) => sum + (item.quantity * item.price), 0);
            const remainingBalance = budget - totalSpent;

            budgetDisplay.textContent = formatCurrency(budget);
            totalSpentDisplay.textContent = formatCurrency(totalSpent);
            remainingBalanceDisplay.textContent = formatCurrency(remainingBalance);
            
            // Muda a cor do saldo com base no valor
            remainingBalanceDisplay.classList.remove('text-green-600', 'text-yellow-600', 'text-red-600');
            if (remainingBalance < 0) {
                remainingBalanceDisplay.classList.add('text-red-600');
            } else if (remainingBalance < budget * 0.1) { // Menos de 10%
                remainingBalanceDisplay.classList.add('text-yellow-600');
            } else {
                remainingBalanceDisplay.classList.add('text-green-600');
            }

            renderItemList();
        };

        /**
         * Renderiza a lista de itens no DOM.
         */
        const renderItemList = () => {
            itemList.innerHTML = '';
            if (items.length === 0) {
                emptyState.classList.remove('hidden');
            } else {
                emptyState.classList.add('hidden');
                items.slice().reverse().forEach(item => {
                    const itemElement = document.createElement('div');
                    itemElement.className = 'bg-white p-4 rounded-xl shadow-sm flex items-center justify-between fade-in';
                    itemElement.dataset.id = item.id;

                    itemElement.innerHTML = `
                        <div>
                            <p class="font-semibold text-gray-800">${item.name}</p>
                            <p class="text-sm text-gray-500">${item.quantity} x ${formatCurrency(item.price)}</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold text-lg">${formatCurrency(item.quantity * item.price)}</p>
                            <button class="remove-item-btn text-xs text-red-500 hover:text-red-700 font-semibold">Remover</button>
                        </div>
                    `;
                    itemList.appendChild(itemElement);
                });
            }
        };
        
        // --- EVENT LISTENERS ---

        setBudgetBtn.addEventListener('click', () => {
            const budgetValue = parseFloat(budgetInput.value);
            if (!isNaN(budgetValue) && budgetValue > 0) {
                budget = budgetValue;
                budgetSection.classList.add('hidden');
                mainApp.classList.remove('hidden');
                updateUI();
            } else {
                alert('Por favor, insira um valor de orçamento válido.');
            }
        });
        
        addItemBtn.addEventListener('click', () => {
            imageInput.click();
        });

        imageInput.addEventListener('change', async (event) => {
            const file = event.target.files[0];
            if (file) {
                // Reset e exibe o modal
                itemPreview.src = URL.createObjectURL(file);
                itemPreview.classList.remove('hidden');
                aiLoader.classList.remove('hidden');
                productNameInput.value = '';
                quantityInput.value = 1;
                priceInput.value = '';
                confirmAddItemBtn.disabled = true;
                addItemModal.classList.remove('hidden');

                // Chama a IA
                try {
                    const base64 = await imageToBase64(file);
                    await callGeminiAPI(base64);
                } catch (error) {
                    console.error("Erro ao processar imagem:", error);
                    aiLoader.classList.add('hidden');
                    productNameInput.value = "Erro ao carregar imagem";
                }

                // Limpa o valor do input para permitir a seleção da mesma foto novamente
                event.target.value = '';
            }
        });

        cancelAddItemBtn.addEventListener('click', () => {
            addItemModal.classList.add('hidden');
        });

        confirmAddItemBtn.addEventListener('click', () => {
            const name = productNameInput.value.trim();
            const quantity = parseInt(quantityInput.value);
            const price = parseFloat(priceInput.value);

            if (name && !isNaN(quantity) && quantity > 0 && !isNaN(price) && price >= 0) {
                const newItem = {
                    id: Date.now(),
                    name,
                    quantity,
                    price
                };
                items.push(newItem);
                updateUI();
                addItemModal.classList.add('hidden');
            } else {
                alert('Por favor, preencha todos os campos corretamente.');
            }
        });

        itemList.addEventListener('click', (event) => {
            if (event.target.classList.contains('remove-item-btn')) {
                const itemDiv = event.target.closest('[data-id]');
                const itemId = parseInt(itemDiv.dataset.id);
                items = items.filter(item => item.id !== itemId);
                updateUI();
            }
        });
    </script>
</body>
</html>
