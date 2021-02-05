<template>
  <q-page class="flex column">

    <header>
      <h1>Прогноз погоды</h1>
      <h2>Тренировочное приложение</h2>
    </header>

    <div class="weather-app">
      <div class="row">
        <div class="form col-lg-3 offset-lg-1 col-md-3 offset-md-1 col-sm-10 offset-sm-1  col-12">
          <q-select
            filled
            :value="model"
            use-input
            hide-selected
            fill-input
            input-debounce="0"
            :options="options"
            @filter="filterFn"
            @input-value="setModel"
            @keyup.enter="getWeatherBySearch" 
            label="Название"
            color="purple"
            label-color="purple"
            :hint="`${this.apiErr}`"
            hint-color="purple"
          >
            <template v-slot:no-option>
              <q-item>
                <q-item-section class="text-grey">
                  No results
                </q-item-section>
              </q-item>
            </template>
          </q-select>

          <q-btn 
            @click="getWeatherBySearch"
            color="purple" 
            label="Искать" 
            icon="search"
          />
        </div>

        <div class="result form col-lg-5 offset-lg-1 col-md-7 offset-md-1 col-sm-10 offset-sm-1 col-12" v-if="weatherData">
          
          <div class="result__city">{{weatherData.city.name}}</div>

          <div class="result__day">
            <div class="day-change">Сегодня</div>
            <img :src="`${apiImg}/${weatherData.list[0].weather[0].icon}@2x.png`" alt="">
            <div class="descr">{{weatherData.list[0].weather[0].description}}</div>
            <div class="temp">{{Math.round(weatherData.list[0].main.temp)}}</div>
          </div>

          <div class="result__day">
            <div class="day-change">Завтра</div>
            <img :src="`${apiImg}/${weatherData.list[8].weather[0].icon}@2x.png`" alt="">
            <div class="descr">{{weatherData.list[8].weather[0].description}}</div>
            <div class="temp">{{Math.round(weatherData.list[8].main.temp)}}</div>
          </div>

          <div class="result__day">
            <div class="day-change">Послезавтра</div>
            <img :src="`${apiImg}/${weatherData.list[16].weather[0].icon}@2x.png`" alt="">
            <div class="descr">{{weatherData.list[16].weather[0].description}}</div>
            <div class="temp">{{Math.round(weatherData.list[16].main.temp)}}</div>
          </div>
        </div>
      </div>
    </div>
    <footer>
      <h2>Приложение прогноза погоды, автор — Пересыпин Федор</h2>
    </footer>   
  </q-page>
</template>

<script>
import { Notify } from 'quasar'
export default {
  name: 'PageIndex',
  data(){
    return{
      weatherData: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/forecast',
      apiKey:'ce77c1cc210b29be3e98700d960f9a76',
      apiImg: 'http://openweathermap.org/img/wn',
      apiErr:'',
      model: '',
      options: this.cityDate,
      cityDate: []
    }
  },
  created() {
    this.getListOfCity()
  },
  methods: {
    getWeatherBySearch(){
      this.apiErr = ''
      this.$axios(`${ this.apiUrl }?q=${ this.model }&appid=${ this.
      apiKey }&units=metric&lang=ru`)
      .then(response => {
          this.weatherData = response.data
      }).catch(error => {
          if (error.response.status === 404) {
            this.apiErr = 'Населенный пункт не найден!'
          }
          if (error.response.status === 400) {
            this.apiErr = 'Напишите что-нибудь!'
          }
          Notify.create({
              message: `${this.apiErr}`,
              color: 'red'
          })
        });
    },

    getListOfCity(){
      this.$axios('https://gist.githubusercontent.com/gorborukov/0722a93c35dfba96337b/raw/435b297ac6d90d13a68935e1ec7a69a225969e58/russia')
      .then((response) => {
        for (let i = 0; i < response.data.length; i++) {
          this.cityDate.push(response.data[i].city)
        }
      }).catch(error => {
          console.log('-----error-------');
          console.log(error)
        })
    },

    filterFn (val, update) {
      update(() => {
        const needle = val.toLocaleLowerCase()
        this.options = this.cityDate.filter(v => v.toLocaleLowerCase().indexOf(needle) > -1)
      })
    },
    setModel (val) {
      this.model = val
    }
  }
}
</script>

<style lang="scss">
  header{
    background:url("../assets/bg2.png") center center/cover no-repeat;
    background-color: rgba(125, 46, 185, 0.45);
    background-blend-mode: luminosity;
    height: 850px;
    color: #FFFFFF;
    h1{
      margin-top: 153px;
      font-weight: 300;
      font-size: 64px;
      line-height: 75px;
      text-align: center; 
    }
    h2{
      font-size: 24px;
      line-height: 28px;
      text-align: center;
    }
  }
  footer{
    height: 264px;
    background: #CBCBCB;
    h2{
      margin-top: 100px;
      font-weight: 300;
      font-size: 12px;
      line-height: 14px;
      text-align: center;
    }
  }

  .weather-app{
    padding: 92px;
    position: absolute;
    min-height: 430px;
    width: 99%;
    top: 480px;
    left: 50%;
    transform: translateX(-50%);
    background: #FFFFFF;
    box-shadow: 0px 8px 10px rgba(0, 0, 0, 0.2), 0px 2px 24px rgba(0, 0, 0, 0.14);
    border-radius: 6px;
  }
  .form{
     color: #000;
    .q-select{
      margin-top: 20px;
      .q-icon{
        font-size: 30px;
      }
      .q-field__messages{
        color: red;
      }
    }
    .q-btn{
      margin-top: 20px;
      border-radius: 30px;
      width: 140px;
      height: 41px;
      font-size: 12px;
      line-height: 14px;
      .q-icon{
        font-size: 12px;
      }
    }
  }
  .error{
    color: red;
  }
  .result{
    &__city{
      margin-top: 20px;
      font-weight: 300;
      font-size: 29px;
      line-height: 34px;
      color: #3C4858;
    }
    &__day{
      height: 50px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 24px;
      line-height: 28px;
      color: #777777;
      .day-change{
        width: 200px;
      }
      img{
        display: block;
      }
      .descr{
        width: 200px
      }
      .temp{
        width: 20px
      }
    }

  }

</style>