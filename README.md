# Comparador de Fans

![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)

## 📋 Sobre o Projeto

O Comparador de Fans é uma aplicação web interativa que permite aos usuários comparar diferentes modelos de ventoinhas (fans) para computador. A ferramenta apresenta dados detalhados sobre especificações como RPM, CFM, pressão e consumo, permitindo comparações lado a lado e destacando o melhor produto para cada necessidade.

## 🛠️ Tecnologias Utilizadas

- **[Vue.js 3](https://vuejs.org/)**: Framework JavaScript progressivo para construção de interfaces de usuário
- **[Vite](https://vitejs.dev/)**: Build tool e servidor de desenvolvimento extremamente rápido
- **[Tailwind CSS](https://tailwindcss.com/)**: Framework CSS utilitário para design rápido e responsivo
- **[PostCSS](https://postcss.org/)**: Ferramenta para transformar CSS com plugins JavaScript
- **[Chart.js](https://www.chartjs.org/)** e **[vue-chartjs](https://vue-chartjs.org/)**: Bibliotecas para criação de gráficos (disponíveis no projeto)

## 🚀 Como Executar o Projeto

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/comparador-de-fans.git

# Entre no diretório
cd comparador-de-fans

# Instale as dependências
npm install

# Execute o servidor de desenvolvimento
npm run dev
```

O aplicativo estará disponível em `http://localhost:3000`.

## 📁 Estrutura do Projeto

```
├── index.html                 # Página HTML principal
├── package.json               # Configurações e dependências do projeto
├── postcss.config.js          # Configuração do PostCSS
├── tailwind.config.js         # Configuração do Tailwind CSS
├── vite.config.js             # Configuração do Vite
└── src/
    ├── App.vue                # Componente Vue principal
    ├── main.js                # Ponto de entrada JavaScript
    ├── assets/
    │   └── main.css           # Estilos CSS globais
    ├── components/
    │   ├── FanComparison.vue  # Componente para comparação de fans
    │   └── FanTable.vue       # Componente para tabela de fans
    └── data/
        └── fans.js            # Dados dos ventiladores para comparação
```

## 📊 Funcionalidades

- Listagem de ventiladores com características técnicas detalhadas
- Filtragem e ordenação por diferentes critérios
- Comparação lado a lado de até dois ventiladores
- Destaque visual para as diferenças e vantagens entre produtos
- Cálculo automático de melhor custo-benefício
- Interface responsiva com modo escuro/claro

## 🔖 Créditos

- **Informações e testes**: [Rockm Gamer](https://www.youtube.com/@rockmgameroficial)
- **Desenvolvimento**: [Mateus Anacleto](https://www.linkedin.com/in/mateus-anacleto-18b548237/)

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

Similar code found with 2 license types
