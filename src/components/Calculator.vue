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
        <a v-if="isLoading" class="button is-warning" @click="calculate">Calculate</a>
        <a v-else class="button is-primary" @click="calculate">Calculating...</a>
      </section>
      <div class="result"></div>
    </div>
    <div class="logs">
      <b-table 
      :data="items">
        <b-table-column field="image" label="Image" width="40" v-slot="props">
          <article class="media">
            <figure class="media-left">
                <p class="image is-64x64">
                  <img :src="props.row.setImageUrl" :alt="props.row.number">
                </p>
            </figure>
        </article>
        </b-table-column>
        <b-table-column field="number" label="Set No" width="40" sortable v-slot="props">
          {{ props.row.number }}
        </b-table-column>
        <b-table-column field="name" label="Set Name" width="40" sortable v-slot="props">
          {{ props.row.name }}
        </b-table-column>
        <b-table-column field="minifiguresCount" label="Minifigures" width="40" sortable numeric v-slot="props">
          {{ props.row.minifiguresCount }}
        </b-table-column>
        <b-table-column field="partsCount" label="Pieces" width="40" sortable numeric v-slot="props">
          {{ props.row.partsCount }}
        </b-table-column>
        <b-table-column field="price" label="Paid Price" width="40" sortable numeric v-slot="props">
          {{ props.row.price }}
        </b-table-column>
        <b-table-column field="income" label="PO Income" width="40" sortable numeric v-slot="props">
          {{ props.row.income }}
        </b-table-column>
        <b-table-column field="incomex" label="Factor" width="40" sortable numeric v-slot="props">
          <b>{{ props.row.incomex }}</b>
        </b-table-column>
        <b-table-column field="actions" label="Actions" width="40" numeric v-slot="props">
          <a target="_blank" :href="props.row.catalogUri">
            <b-icon
                pack="fas"
                icon="money-bill-alt">
            </b-icon>
          </a>
          <a target="_blank" :href="props.row.inventeroUri">
            <b-icon
                pack="fas"
                icon="puzzle-piece">
            </b-icon>
          </a>
        </b-table-column>
      </b-table>
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
        incomex: null,
        catalogUri: null,
        inventeroUri: null,
        setImageUrl: null,
        minifiguresCount: null,
        partsCount: null
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
            "https://brickton-api.netlify.app/.netlify/functions/api/po/" +
              this.number +
              "/" +
              this.price
          )
          .then(getResponse => {
            const data = getResponse.data[0]
            this.items.push({
              number: this.number,
              price: this.round(this.price, 100),
              name: data.setName,
              income: this.round(data.try.sixMonth, 100),
              incomex: data.calculation.sixMonthFactor,
              catalogUri: data.bricklink.catalogUri,
              inventeroUri: data.bricklink.inventeroUri,
              setImageUrl: data.brickset.set.set_img_url,
              minifiguresCount: data.brickset.minifigures.count,
              partsCount: data.brickset.set.num_parts
            });
            this.isLoading = true;
            this.number = null;
            this.price = null;
          })
          .catch(function(error) {
            console.log("HATA " + error);
          });
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
