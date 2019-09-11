<template>
  <div class="ui container">
    <vuetable
      ref="vuetable"
      :fields="fields"
      :api-url="api_path"
      :css ="css.table"
      :sort-order ="sortOrder"
      :show-sort-icons="true"
      :per-page="perPage"
      data-path="links.data"
      pagination-path="links.pagination"
      detail-row-component="my-detail-row"
      @vuetable:cell-clicked="onCellClicked"
      @vuetable:pagination-data="onPaginationData">
      </vuetable>
    <vuetable-pagination ref="pagination" :css="css.pagination"  
    @vuetable-pagination:change-page="onChangePage"></vuetable-pagination>
  </div>
</template>
     
<script>
import Vue from "vue";
import Vuetable from "vuetable-2/src/components/Vuetable";
import VuetablePagination from "vuetable-2/src/components/VuetablePagination";
import CssConfig from "./VuetableCssConfig.js";
import { API_PATH_MD_TABLE_ALL, API_PATH_MD_TABLE } from "@/utils/variables.js";
import axios from "axios";
import moment from "moment";


export default {
  components: {
    Vuetable,
    VuetablePagination,

  },
  data() {
    return {
      api_path: API_PATH_MD_TABLE_ALL,
      perPage: 3,
      mdtables: [],
      allData: false,
      currentPage: 1,
      css: CssConfig,
           fields: [
        { name: "id", visible: false },
        {
          name: "XX",
          title: "XX",
          sortField: "Storage Type",
          titleClass: "center aligned",
          dataClass: "center aligned"
        },
        {
          name: "YY",
          title: "YY",
          sortField: "YY",
          dataClass: "center aligned",
          titleClass:"center aligned"
        },
        {
          name: "ZZ",
          title: "ZZ",
          titleClass:"center aligned",
          sortField: "ZZ",
          dataClass: "center aligned",
        },
      ],
      sortOrder: [{ field: "id", direction: "desc" }]
    };
  },
  mounted() {
      //
    })

  },
  created() {
      bus.$on("loadTable1", comp => {
      this.refreshtables(comp);
    })
  },

  methods: {
    refreshtables(value) {
      this.api_path = API_PATH_MD_TABLE_ALL;
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
    },
    formatDate(value, fmt = "D MMM YYYY") {
      return value == null ? "" : moment(value, "YYYY-MM-DD").format(fmt);
    }
    },
    onPaginationData(paginationData) {
      console.log("pased here", paginationData);
      this.$refs.pagination.setPaginationData(paginationData);
    },
    onChangePage(page) {
      var self = this;
      console.log("onChange pagination page", page);
      this.$refs.vuetable.changePage(page);
    },

    onCellClicked(data, field, event) {
      console.log("Cell clicked", data);
      console.log("Cell clicked", data["Table Name"]);
      bus.$emit("loadSecondTable", data["Table Name"]);
    }
      
  }
};
</script>
