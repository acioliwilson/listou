<template>
    <MainHeader />
    <router-view />
</template>

<script>
import MainHeader from './components/MainHeader.vue'
import * as XLSX from "xlsx";

export default {
    components: {
        MainHeader,
    },
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
