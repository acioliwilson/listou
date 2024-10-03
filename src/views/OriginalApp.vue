<template>
    <div class="container">
      <h1>Lista de Compras</h1>
  
      <!-- Botão para upload de uma planilha -->
      <div class="mb-3">
        <h3>Importar Planilha</h3>
        <input type="file" @change="handleFileUpload" />
      </div>
  
      <hr />
  
      <!-- Formulário para adicionar novo item manualmente -->
      <div class="input-group mb-3">
        <input
          v-model="newItem"
          type="text"
          class="form-control"
          placeholder="Novo item"
        />
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
        <button class="btn btn-primary" @click="addItem">Adicionar</button>
      </div>
  
      <!-- Mensagem de erro -->
      <div v-if="errorMessage" class="alert alert-danger">
        {{ errorMessage }}
      </div>
  
      <hr />
  
      <!-- Exibir os itens por categorias -->
      <section>
        <div v-for="(categoryItems, category) in groupedItems" :key="category">
          <h3>{{ category }}</h3>
          <ul class="list-group mb-3">
            <li
              v-for="item in categoryItems"
              :key="item.name"
              class="list-group-item d-flex justify-content-between align-items-center"
            >
              <div>
                <input type="checkbox" v-model="item.completed" />
                <span :class="{ 'text-decoration-line-through': item.completed }">
                  {{ item.name }}
                </span>
              </div>
              <div v-if="item.completed">
                <input
                  v-model.number="item.price"
                  type="number"
                  placeholder="Preço"
                  class="form-control"
                />
              </div>
              <button class="btn btn-danger" @click="removeItem(item.name)">
                Remover
              </button>
            </li>
          </ul>
        </div>
      </section>
      <section v-if="!items.length">
        <p>Lista Vazia</p>
      </section>
  
      <hr />
  
      <!-- Total de itens e valor total -->
      <div class="mt-4">
        <h4>Total de Itens: {{ totalItems }}</h4>
        <h4>Valor Total: R$ {{ totalPrice.toFixed(2) }}</h4>
      </div>
    </div>
  </template>
  
  <script>
  import * as XLSX from "xlsx";
  
  export default {
    data() {
      return {
        newItem: "", // Armazena o nome do novo item manual
        newCategory: "", // Armazena a categoria do novo item manual
        items: [], // Armazena os itens, seja da planilha ou inseridos manualmente
        file: null, // Armazena o arquivo de planilha para upload
        errorMessage: "", // Exibe mensagem de erro se necessário
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
    },
    methods: {
      // Manipula o arquivo selecionado
      handleFileUpload(event) {
        const file = event.target.files[0];
        if (!file) return;
  
        const reader = new FileReader();
  
        reader.onload = (e) => {
          const data = new Uint8Array(e.target.result);
          const workbook = XLSX.read(data, { type: "array" });
  
          // Seleciona a primeira planilha
          const firstSheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[firstSheetName];
  
          // Converte a planilha para JSON
          const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
  
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
    },
  };
  </script>
  
  <style scoped>
  .text-decoration-line-through {
    text-decoration: line-through;
  }
  </style>
  