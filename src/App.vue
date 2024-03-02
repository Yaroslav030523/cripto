<template class="">
<div  class="bgImg mainContaimer" >

<body class="container" >
<div class="">
  <div class="">
  </div>
  <div class="">
    <section>
      <div class="flex">
        <div class="max-w-xs">
          <label for="wallet" class="neonText block text-sm font-medium text-gray-700"
            >Тікер</label
          >
          <div class="displayBlock mt-1 relatives rounded-md shadow-md">
            <div 
            @mouseenter="hintVision = true"
            @mouseleave="hintVision = false" 
            class="inputContainer" >
            <input
            @keyup="hintAdd(), clinerSaport()"
            @keyup.enter=" tickerCheck()"
            
            @focus="hintVision = true"
              v-model="ticker"
              type="text"
              name="wallet"
              id="wallet"
              class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
              placeholder="Наприклад BTC"
            />
            <div v-if="hintVision" class="hintContainer">
<div class="flex hint"
v-for="(elem, index) in testTextFilter "
:key="index"
@click="this.ticker = elem, UniquenessСheck() "
@wheel.prevent="skrolHint()"
> <b>{{ elem }} </b> 
</div>
</div>

</div>
            <div v-if="chekInclud" class="textSm flicker-in-1 text-red-600">Такий тікер вже є!</div>
          <div v-if="chekIncExist" class="textSm flicker-in-1 text-red-600">Тікер не знайдено! </div>
          </div>


        </div>
      </div>
      <button
      @click=" tickerCheck()"
        type="button"
        class="buttunAdd slide-in-bck-center my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
      >
        <!-- Heroicon name: solid/mail -->
        добавити
      </button>

    </section>

      <hr v-if="tickers.length" class="w-full border-t border-gray-600 my-4" />
    <div> <p class="neonText" >Фільтр</p> <input v-model="filter" input="page = 1"/>   </div>

    <button 
    class="buttonNext "
    @click=" page = page + 1"
    v-if="hasNextPage"
    >Вперед</button> 

    <button 
    class="buttonNext"
    @click="page = page - 1"
    v-if="page > 1"
    > Назад</button>

    <hr v-if="tickers.length" class="w-full border-t border-gray-600 my-4" />
      <div class=" mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3 flex-ya">
        <div
        v-for="t in filteredTickers()"
        :key="t.name"
        @click="select(t)"
        :class="{'wrapper' : sel === t  }"
          class="blocks cursor-pointer"
        >
          <!-- <div class="px-4 py-5 sm:p-6 text-center"> -->
            <div>
            <div class="text-sm font-medium text-gray-500 truncate">
            {{ t.name }} - USD
            </div>
            <div class="textCenter mt-1 text-3xl font-semibold text-gray-900">
              {{ t.price }}
            </div>
          </div>
          <div class="w-full border-t border-gray-200"></div>
          <button
          @click.stop="hendleDelete(t)"
            class="buttonshower flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
          >
              видалити
                      </button>
        </div>

      </div>
      <hr v-if="tickers.length" class="w-full border-t border-gray-600 my-4" />

    <section v-if="sel" class="relative flex-ya">
      <div>
      <h3 class="selText text-lg leading-6 font-medium text-gray-900 my-8 ">
      {{sel.name}} - USD
      </h3>
      <button 
      @click="sel = null"
        type="button"
        class="buttonshower"
      >
      закрити
      </button>
    </div>
      <div class="flex items-end border-gray-600 border-b border-l h-64 flex-ya graph-container">
        <div v-for="(bar, idx) in normalizeGraph()"
        :key="idx"
        :style="{height: `${bar}%`}"
          class="bg-purple-800 border w-10 h-24 graph-div"
        ></div>
  
      </div>

    </section>

  </div>
</div>
</body>
</div>

</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      key: '545b10b9314218383f1c62d317f8792cb16b4fd5306d14daa1982a2e4bffb30a',
      ticker: "",
      tickers: [],
      sel: null,
      visible: false,
      chekInclud: false,
      chekIncExist: false,
      graph: [],
      page: 1,
      filter: '',
      hasNextPage: true,
      arr:[],
      exchangeTsymValues:[],
      tsymUnics : [],
      cryptocurrencies: [],
      hint: [1,2,3],
      testTextFilter: '',
      hintVision: false
    };
  },
  created() {
    const tickersData = localStorage.getItem("cryptonomicon-list");
    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach(ticker => {
        this.fetchAndUpdateTickerData(ticker.name);
      });
    }
  },
    mounted() {
    fetch(`https://min-api.cryptocompare.com/data/v4/all/exchanges`)
    .then(response => response.json())
  .then(data => {
    if (data.Data && data.Data.exchanges && data.Data.exchanges.ascendex) {
      this.cryptocurrencies = Object.keys(data.Data.exchanges.ascendex.pairs);
      // console.log(this.cryptocurrencies); // Масив із списком усіх криптовалют
    } else {
      console.error('Неможливо знайти криптовалют');
    }
  })
  .catch(error => console.error('Помилка:', error));

  },
  methods: {
   
    skrolHint() {
      const event = window.event || event;

      if (event && event.deltaY !== undefined) {
        let direction = event.deltaY > 0 ? 'down' : 'up';
        if (direction === 'down') {
          let firstElement = this.testTextFilter.shift();
          this.testTextFilter.push(firstElement); 
        } else {
          let lastElement = this.testTextFilter.pop();
          this.testTextFilter.unshift(lastElement);
        }
      }
    
  },
    clinerSaport(){
      this.chekInclud = false,
      this.chekIncExist = false
    },
    filling(){
      console.log(this.ticker);  
        console.log(this.elem)
    },
    hintAdd(){
      if(this.ticker.length > 0){
    this.testTextFilter = this.cryptocurrencies.filter(item => item.toString().includes(this.ticker.toUpperCase()))
    console.log( this.testTextFilter)      
      } else {
        this.testTextFilter = ''
      }
    },
    tickerCheck(){
      
      this.ticker = this.ticker.toUpperCase()
    let res = this.cryptocurrencies.some( elem => elem == this.ticker)
    if (res){  console.log('tickerCheck()= знайдено')
    this.UniquenessСheck() }
    else{
      this.chekIncExist = true;
      console.log('tickerCheck()= НЕ знайдено')
    }
    
    },

    UniquenessСheck() { 
    console.log('ticker=' + this.ticker);
    let found = false;

    for (let i = 0; i < this.tickers.length; i++) {
      console.log('this.tickers[i].name=' + this.tickers[i].name);
      if (this.tickers[i].name === this.ticker) {
        found = true;
        break;
      }
    }
    if (found == true) {
      this.chekInclud = true;
      console.log('UniquenessСheck=false');
    } else {
      this.add();
      this.testTextFilter =''

      console.log('UniquenessСheck=TRUE');
  
}
},

    filteredTickers() {
      let start = (this.page - 1) * 6;
      let end = this.page * 6;

      let filteredTickers = this.tickers.filter(ticker => 
        ticker.name.includes(this.filter.toUpperCase())
      );

      this.hasNextPage = filteredTickers.length > end;

      return filteredTickers.slice(start, end);
    },
    select(ticker) {
      this.sel = ticker;
      this.graph = [];
    },
    add() {
      
      const currentTicker = {
        name: this.ticker,
        price: "-"
      };
      if (currentTicker.name.trim() !== ' ') {
        this.tickers.push(currentTicker);
        this.filter = '';
        localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
        this.fetchAndUpdateTickerData(currentTicker.name);
        // this.ticker = "";
      }
    },
    fetchAndUpdateTickerData(tickerName) {
      setInterval(async () => {
        const f = await fetch(`https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=${this.key}`);
        const data = await f.json();
        const foundTicker = this.tickers.find(t => t.name === tickerName);
        if (foundTicker) {
          foundTicker.price = data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);
          if (this.sel?.name === tickerName) {
            this.graph.push(data.USD);
            if (this.graph.length > 10) {
              this.graph.shift();
            }
          }
        }
      }, 3000);
      this.ticker = "";
    },
    hendleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter(t => t !== tickerToRemove);
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
    },
    normalizeGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);
      
      return this.graph.map(
        price => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },
  }
}
</script>


<style >
@import url('./AppCss.css');

</style>
