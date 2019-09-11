<template>
  <div class="ui container">
    <vuetable
      ref="vuetable"
      :fields="fields"
      :api-url="api_path"
      :css ="css.table"
      :multi-sort ="true"
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
import { bus } from "../main.js";
import moment from "moment";


export default {
  components: {
    Vuetable,
    VuetablePagination,

  },
  data() {
    return {
      tbDomain:"",
      
      api_path: API_PATH_MD_TABLE_ALL,
      perPage: 3,
      mdtables: [],
      allData: false,
      currentPage: 1,
      css: CssConfig,
           fields: [
        { name: "id", visible: false },
        {
          name: "Storage Type",
          title: "Storage Type",
          sortField: "Storage Type",
          titleClass: "center aligned",
          dataClass: "center aligned"
        },
        {
          name: "Source System",
          title: "Source System",
          sortField: "Source System",
          dataClass: "center aligned",
          titleClass:"center aligned"
        },
        {
          name: "Table Name",
          title:
            '<span class="orange glyphicon glyphicon-user"></span> Table Name',
          dataClass: "center aligned",
          titleClass:"center aligned"
          
        },
        {
          name: "Table Description",
          title: "Table Description",
          titleClass:"center aligned",
          sortField: "Table Description",
          dataClass: "center aligned",
        },
        {
          name: "Row Count",
          title: "Row Count",
          sortField: "Row Count",
          titleClass: "center aligned", 
          dataClass: "center aligned"

        },
        {
          name: "Data As Of Date",
          title: "Data As Of Date",
          callback: "formatDate|YYYY-MM-DD",
          sortField: "Data As Of Date",
          titleClass: "center aligned", 
          dataClass: "center aligned"
        },
        {
          name: "Load Date",
          title: "Updated Date",
          callback: "formatDate|YYYY-MM-DD",
          sortField: "Load Date",
          titleClass: "center aligned", 
          dataClass: "center aligned"
        },
        {
          name: "Data Domain",
          title: "Data Domain",
          callback: "displayDomain",
          titleClass: "center aligned", 
          dataClass: "center aligned"
        }
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
    }),
      bus.$on("loadTableData2", comp => {
        this.searchTable(comp);
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
    
    getTableDataByName(value) {
      console.log(value);
      this.api_path = API_PATH_MD_TABLE + "searchtable/" + value;
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
    },
    formatDate(value, fmt = "D MMM YYYY") {
      return value == null ? "" : moment(value, "YYYY-MM-DD").format(fmt);
    },
    changeUrl(value) {
      this.api_path = API_PATH_MD_TABLE + "search/" + value;
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
    },
    getMdTables() {
      var self = this;
      axios
        .get(API_PATH_MD_TABLE_ALL)
        .then(function(response) {
          var mdtables = response.data;
          var mdtables = JSON.parse(JSON.stringify(response.data));
          console.log("medtables", mdtables);
          self.mdtables = mdtables;
          //pagination = self.$refs.vuetable.makePagination(response.length);

          console.log("medtables1", self.mdtables);
        })
        .catch(function(/*error*/) {
          // handle error
          // console.log("Get error", error);
          console.log("error");
        })
        .then(function() {
          self.loading = false;s
        });
    },
    pronacLabel(value) {
      return '<p class="ui teal label">' + value + "</p>";
    },
    responsavelLabel(value) {
      return value != " " ? value : "null";
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
