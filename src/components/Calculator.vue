<template>
  <div class="container">
    <div class="notification">
      <section>
        <b-field label="Set Number">
          <b-input ref="number" v-model="number"></b-input>
        </b-field>
        <b-field label="Price">
          <b-input v-model="price" @keyup.enter.native="calculate"></b-input>
        </b-field>
        <a v-if="isLoading" class="button is-warning" @click="calculate">Hesapla</a>
        <a v-else class="button is-primary" @click="calculate">Hesaplanıyor...</a>
      </section>
      <div class="result"></div>
    </div>
    <div class="logs">
      <div class="columns is-multiline is-mobile">
        <div class="column is-half" v-for="item in items.slice().reverse()">
          <div class="box">
            <article class="media">
              <div class="media-content">
                <div class="content">
                  <p>
                    <strong>{{ item.number }}</strong>
                    <small>{{ item.name }}</small>
                    <br>
                    <small>
                      {{ item.price }} Ödendi -
                      {{ item.income }} Kazanılacak
                    </small>
                    <br>
                    <strong>x{{ item.incomex }}</strong>
                    <small> +{{ round((item.income - item.price),100) }} TL</small>
                  </p>
                </div>
              </div>
            </article>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      number: null,
      isLoading: true,
      price: null,
      set: {
        number: null,
        price: null,
        name: null,
        income: null,
        incomex: null
      },
      items: []
    };
  },
  methods: {
    round(value, round) {
      return Math.round(value * round) / round;
    },
    retrieve: async function() {
      try {
        await this.axios
          .get(
            "https://basarozcan.com/brickton/calc/newapi.php?set=" +
              this.number +
              "&price=" +
              this.price +
              "&currency=TRY"
          )
          .then(getResponse => {
            this.items.push({
              number: this.number,
              price: this.round(this.price, 100),
              name: getResponse.data.data.title,
              income: this.round(getResponse.data.try.sixmavg, 100),
              incomex: this.round(
                getResponse.data.try.sixmavg / this.price,
                1000
              )
            });
            this.isLoading = true;
            this.number = null;
            this.price = null;
          })
          .catch(function(error) {
            console.log("HATA " + error);
          });
        //http://basarozcan.com/brickton/calc/mini.php?set=75040&price=100&currency=TRY
      } catch (error) {
        alert(error);
      }
    },

    calculate() {
      this.isLoading = false;
      this.retrieve();
      this.$refs.number.$el.children[0].focus();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.logs {
  margin: 10px;
}
</style>
