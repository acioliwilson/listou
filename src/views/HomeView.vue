<template>  
    <section class="form">
        <div class="container">
            <div class="d-flex align-items-center justify-content-center gap-2">
                <label for="file" class="file-upload-btn">
                    <i class="bi bi-cloud-upload"></i>
                </label>
                <input type="file" id="file" @change="handleFileUpload" />
                <div class="input-group mb-3">
                    <input v-model="newItem" type="text" class="form-control" placeholder="Novo item" />
                    <select v-model="newCategory" class="form-select">
                        <option disabled value="">Escolha uma categoria</option>
                        <option value="Alimentos">Alimentos</option>
                        <option value="Frios">Frios</option>
                        <option value="Carnes">Carnes</option>
                        <option value="Limpeza">Limpeza</option>
                        <option value="Bebidas">Bebidas</option>
                        <option value="Hortifruti">Hortifruti</option>
                        <option value="Padaria">Padaria</option>
                        <option value="Laticínios">Laticínios</option>
                        <option value="Doces">Doces</option>
                        <option value="Cereais">Cereais</option>
                        <option value="Higiene Pessoal">Higiene Pessoal</option>
                        <option value="Congelados">Congelados</option>
                        <option value="Pet Shop">Pet Shop</option>
                    </select>
                    <button class="btn btn-primary2" @click="addItem">Adicionar</button>
                </div>
            </div>
        </div>
    </section>
    
    <!-- Mensagem de erro -->
    <div class="container error" v-if="errorMessage">
        <div class="alert alert-danger">
            {{ errorMessage }}
        </div>
    </div>
    
    <!-- Exibir os itens por categorias -->
    <section class="container mb-8 items-wrapper" v-if="!hide">
        <div v-for="(categoryItems, category) in groupedItems" :key="category" class="items">
            <h3>{{ category }}</h3>
            <ul class="list-group mb-3">
                <li v-for="item in categoryItems" :key="item.name" class="list-group-item d-flex justify-content-between align-items-center">
                    <div>
                        <input type="checkbox" id="check" v-model="item.completed" />
                        <span :class="{ 'text-decoration-line-through': item.completed }" id="item-name">
                            {{ item.name }}
                        </span>
                    </div>
                    <div v-if="item.completed">
                        <input v-model.number="item.price" type="number" placeholder="Preço" class="form-control" />
                    </div>
                    <button class="btn btn-danger" @click="removeItem(item.name)">
                        Remover
                    </button>
                </li>
            </ul>
        </div>
    </section>
    <section v-if="!items.length" class="container">
        <p class="text-center my-4">Lista Vazia</p>
    </section>
    
    <section class="total" v-if="hide">
        <div class="container">
            <button class="back" @click="back">
                <i class="bi bi-arrow-left"></i>
            </button>
            <div class="d-flex justify-content-between">
                <div class="totalItems">
                    <h4>Itens:</h4>
                    <h1>{{ (( checkout.totalItens < 10 )?'0'+checkout.totalItens : checkout.totalItens) }}</h1>
                </div>
                <div class="totalPrice">
                    <h4>Total:</h4>
                    <h1>R$ {{ checkout.totalValor ? checkout.totalValor.toFixed(2) : '0.00' }}</h1>
                </div>
            </div>
            <p class="alert alert-info" role="alert">
                Você selecionou {{ checkout.totalItens }} {{ (checkout.totalItens > 1 ? 'itens' : 'item') }} com um valor total de R$ {{ checkout.totalValor ? checkout.totalValor.toFixed(2) : '0.00' }}.
            </p>
        </div>
    </section>
    
    <hr class="my-3">
    
    <footer>
        <button id="checkout" @click="CheckOut">
            <i class="bi bi-check-lg"></i>
        </button>
    </footer>
    </template>
    
    <script>
    import * as XLSX from "xlsx";
    
    export default {
        data() {
            return {
                newItem: "",
                newCategory: "",
                items: [],
                file: null,
                errorMessage: "",
                hide: false,
                checkout: {
                    selectedItemsCount: '',
                    totalPrice: '',
                },
            };
        },
        computed: {
            // Agrupa os itens por categorias
            groupedItems() {
                return this.items.reduce((groups, item) => {
                    const category = item.category || "Sem Categoria";
                    if (!groups[category]) {
                        groups[category] = [];
                    }
                    groups[category].push(item);
                    return groups;
                }, {});
            },
            // Calcula o total de itens (concluídos ou não)
            totalItems() {
                return this.items.length;
            },
            // Calcula o valor total dos itens concluídos
            totalPrice() {
                return this.items.reduce((total, item) => {
                    return item.completed ? total + (item.price || 0) : total;
                }, 0);
            },
            // Conta quantos itens foram selecionados (concluídos)
            selectedItemsCount() {
                return this.items.filter(item => item.completed).length;
            },
        },
        methods: {
            // Manipula o arquivo selecionado
            handleFileUpload(event) {
                const file = event.target.files[0];
                if (!file) return;
    
                const reader = new FileReader();
    
                reader.onload = (e) => {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {
                        type: "array"
                    });
    
                    // Seleciona a primeira planilha
                    const firstSheetName = workbook.SheetNames[0];
                    const worksheet = workbook.Sheets[firstSheetName];
    
                    // Converte a planilha para JSON
                    const jsonData = XLSX.utils.sheet_to_json(worksheet, {
                        header: 1
                    });
    
                    // A primeira linha contém os cabeçalhos
                    const headers = jsonData[0];
    
                    // Encontra os índices das colunas "Item" e "Categoria"
                    const itemIndex = headers.findIndex((header) =>
                        header.toLowerCase().includes("item")
                    );
                    const categoryIndex = headers.findIndex((header) =>
                        header.toLowerCase().includes("categoria")
                    );
    
                    if (itemIndex === -1 || categoryIndex === -1) {
                        alert('Colunas "Item" ou "Categoria" não encontradas.');
                        return;
                    }
    
                    // Mapeia os dados das linhas restantes
                    this.items = [
                        ...this.items,
                        ...jsonData.slice(1).map((row) => ({
                            name: row[itemIndex] || "Sem nome",
                            category: row[categoryIndex] || "Sem Categoria",
                            completed: false,
                            price: 0,
                        })),
                    ];
                };
    
                // Lê o arquivo
                reader.readAsArrayBuffer(file);
            },
            // Adicionar um novo item manualmente
            addItem() {
                if (this.newItem === "" || this.newCategory === "") {
                    this.errorMessage = "Por favor, preencha o nome e a categoria do item.";
                    return;
                }
    
                this.items.push({
                    name: this.newItem,
                    category: this.newCategory,
                    completed: false,
                    price: 0,
                });
    
                // Limpa os campos
                this.newItem = "";
                this.newCategory = "";
                this.errorMessage = "";
            },
            // Remover item
            removeItem(name) {
                this.items = this.items.filter((item) => item.name !== name);
            },
            CheckOut() {
                // Recalcular a quantidade de itens selecionados e o total antes de atualizar a interface
                const totalItens = this.items.filter(item => item.completed).length;
                const totalValor = this.items.reduce((total, item) => {
                    return item.completed ? total + (item.price || 0) : total;
                }, 0);
    
                // Verifica se existem itens completados
                if (totalItens === 0) {
                    this.errorMessage = "Nenhum item foi selecionado.";
                    return;
                }
    
                // Atualiza os valores do checkout
                this.checkout.totalItens = totalItens;
                this.checkout.totalValor = totalValor;
    
                // Alterna a exibição do total
                this.hide = !this.hide;
            },
            back() {
                this.hide = !this.hide;
            }
        },
    };
    </script>
    
    <style lang="css" scoped>
    header {
        width: 100%;
        max-width: 500px;
        background-color: #F8772F;
        background-image: url('@/assets/header-background.png');
        background-position-x: right;
        background-repeat: no-repeat;
        background-size: cover;
        padding: 45px 0;
        display: table;
        margin: 0 auto;
    }
    
    .logo {
        font-size: 35px;
        font-weight: 600;
        color: #FFF;
        font-family: 'Neo Sans Std', sans-serif;
        display: flex;
        gap: 7px;
        align-items: center;
    }
    
    .logo i {
        font-size: 30px;
    }
    
    section.form {
        margin: 30px auto;
        display: table;
        max-width: 500px;
    }
    
    section.form #file {
        display: none;
    }
    
    section.form .file-upload-btn {
        cursor: pointer;
        background: #F8772F;
        color: #FFF;
        font-size: 18px;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 45px;
        height: 43px;
        border-radius: 15px;
        line-height: 1;
        transition: .3s ease all;
        margin-top: -15px;
    }
    
    section.form .file-upload-btn i {
        margin-top: 3px;
    }
    
    section.form .file-upload-btn:hover {
        background: #3ddd58;
    }
    
    .btn-primary2 {
        background: #F8772F;
        color: #FFF;
    }
    
    .btn-primary2:hover {
        background: #3ddd58;
        color: #FFF;
    }
    
    .items-wrapper {
        max-width: 500px !important;
        margin: 0 auto;
    }
    
    .items h3 {
        font-family: 'Neo Sans Std', sans-serif;
        font-size: 25px;
        font-weight: 600;
        color: #414141;
    }
    
    #check {
        margin-right: 7px;
    }
    
    .done {
        font-size: 18px;
        margin-right: 7px;
    }
    
    #item-name {
        font-size: 18px;
        color: #414141;
        font-family: 'Roboto Mono', monospace;
    }
    
    input[type=number] {
        width: 70px;
    }
    
    hr {
        max-width: 500px !important;
        margin: 0 auto;
    
    }
    
    footer {
        width: 100%;
        max-width: 500px;
        margin: 0 auto;
        height: 100px;
        display: flex;
        justify-content: center;
        background: #f75437;
        position: fixed;
        bottom: 0;
        left: 50%;
        transform: translateX(-50%) translateY(30px);
        box-shadow: 0 -5px 30px rgba(0, 0, 0, .1);
    }
    
    footer button {
        border: 3px solid #FFF;
        background: #F8772F;
        color: #FFF;
        font-size: 25px;
        border-radius: 900px;
        width: 60px;
        height: 60px;
        display: flex;
        justify-content: center;
        align-items: center;
        transform: translateY(-25px);
        transition: .3s ease all;
    }
    
    footer button:hover {
        background: #3ddd58;
    }
    
    .mb-8 {
        margin-bottom: 100px;
    }
    
    .error {
        max-width: 500px !important;
    }
    
    .total {
        width: 100%;
        max-width: 500px;
        margin: 0 auto;
        display: table;
    }
    
    button.back {
        cursor: pointer;
        background: #F7F7F7;
        border: 1px solid #EAEAEA;
        border-radius: 900px;
        display: flex;
        margin-bottom: 30px;
        width: 40px;
        height: 40px;
        justify-content: center;
        align-items: center;
        font-family: 'Roboto Mono', monospace;
        font-weight: 600;
        font-size: 1rem;
    }
    
    .totalItems {
        font-family: 'Neo Sans Std', sans-serif;
    }
    .totalItems h4 {
        font-size: 22px;
        font-weight: 600;
    }
    .totalItems h1 {
        font-size: 35px;
        font-weight: 800;
    }
    
    .totalPrice {
        font-family: 'Neo Sans Std', sans-serif;
        text-align: right;
    }
    .totalPrice h4 {
        font-size: 22px;
        font-weight: 600;
    }
    .totalPrice h1 {
        font-size: 35px;
        font-weight: 800;
    }
    </style>
    