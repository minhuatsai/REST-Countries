<template>
  <div class="home text-left">
    <div class="search-wrapper my-3 px-2">
      <input
        type="text"
        class="input-filter-country form-control"
        placeholder="Filter country..."
        v-model="filterCountry"
      />
    </div>
    <b-table
      striped
      :items="getTableItems"
      :fields="fields"
      :filter="filterCountry"
      :per-page="restCountriesTablePerPage"
      :current-page="restCountriesTableCurrentPage"
      @filtered="afterFilterTable"
      class="rest-countries-table"
    >
      <template v-slot:cell(flag)="data">
        <span v-html="data.value"></span>
      </template>
      <template v-slot:cell(name)="data">
        <b-button v-b-modal @click="showCountryMoreInfo(data.index)">{{
          data.value
        }}</b-button>
      </template>
    </b-table>
    <b-pagination
      align="right"
      v-model="restCountriesTableCurrentPage"
      :total-rows="restCountriesTableRows"
      :per-page="restCountriesTablePerPage"
    ></b-pagination>
    <b-modal
      ref="country-moreinfo"
      scrollable
      ok-only
      :title="
        restCountriesData[restCountriesActiveIndex] &&
        restCountriesData[restCountriesActiveIndex].name
      "
    >
      <b-row
        v-for="(dataItem, dataItemKey) in restCountriesData[
          restCountriesActiveIndex
        ]"
        :key="dataItem.name"
      >
        <b-col class="font-weight-bold">{{ dataItemKey }} </b-col>
        <b-col>
          <template v-if="dataItemKey === 'flag'">
            <img :src="dataItem" class="flag-icon" />
          </template>
          <template v-else>
            {{ dataItem }}
          </template>
        </b-col>
      </b-row>
    </b-modal>
  </div>
</template>

<script>
//get Rest Countries
const getRestCountries = (callback) => {
  fetch("https://restcountries.eu/rest/v2/all")
    .then((rsp) => {
      return rsp.json();
    })
    .then((rsp) => {
      callback(rsp);
    });
};

export default {
  name: "Home",
  components: {},
  data() {
    return {
      restCountriesData: [],
      restCountriesActiveIndex: Number,
      restCountriesTablePerPage: 25,
      restCountriesTableCurrentPage: 1,
      restCountriesTableRows: 0,
      filterCountry: "",
      fields: [
        {
          key: "flag",
          label: "國旗",
        },
        {
          key: "name",
          label: "國家名稱",
          sortable: true,
        },
        {
          key: "alpha2Code",
          label: "國家代碼",
          sortable: false,
        },
        {
          key: "alpha3Code",
          label: "國家代碼",
          sortable: false,
        },
        {
          key: "nativeName",
          label: "母語名稱",
          sortable: false,
        },
        {
          key: "altSpellings",
          label: "替代國家名稱",
          sortable: false,
        },
        {
          label: "國際電話區號",
          sortable: false,
        },
      ],
      tableItems: [],
    };
  },
  methods: {
    setTable: function (data) {
      data.forEach((dataItem) => {
        const tableItemsOption = {};
        this.fields.forEach((fieldItem, fieldItemIndex) => {
          const { key } = fieldItem;
          if (key === "name") {
            this.fields[fieldItemIndex].tdClass = "name-field";
          }
          if (key === "flag") {
            tableItemsOption[
              key
            ] = `<img src="${dataItem[key]}" class="flag-icon" />`;
            this.fields[fieldItemIndex].tdClass = "flag-field";
          } else {
            tableItemsOption[key] = dataItem[key];
          }
        });
        this.tableItems.push(tableItemsOption);
      });
    },
    showCountryMoreInfo: function (rowIndex) {
      this.restCountriesActiveIndex = rowIndex;
      this.$refs["country-moreinfo"].show();
    },
    afterFilterTable(filteredItems) {
      this.restCountriesTableRows = filteredItems.length;
      this.currentPage = 1;
    },
  },
  computed: {
    getTableItems() {
      const filtered = this.tableItems.filter((tableItem) => {
        console.log(tableItem.name.toLowerCase());
        return tableItem.name.toLowerCase().includes(this.filterCountry);
      });
      return filtered;
    },
  },
  created() {
    getRestCountries((data) => {
      this.restCountriesData = data;
      this.restCountriesTableRows = this.restCountriesData.length;
      this.setTable(data);
    });
  },
};
</script>
<style lang="scss">
img.flag-icon {
  max-width: 30px;
}
.home {
  .input-filter-country {
    max-width: 300px;
  }
  .rest-countries-table {
    .name-field {
      button {
        width: 150px;
      }
    }
  }
}
</style>
