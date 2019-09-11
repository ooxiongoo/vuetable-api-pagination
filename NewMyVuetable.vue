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
      
           <div slot="actions" scope="props">
        <div class="custom-actions">
           <button class="ui basic button"
            @click="onActionEdit('edit-item', props.rowData)" >
            <i class="edit icon"></i>
              <!-- <input type="text" v-model="label" v-if="props.rowData.edit" 
           @keyup.enter="onActionEdit(props.rowData)" @close-desc="onActionEdit(props.rowData)"> -->
            </button>
          

          <button class="ui basic button"
            @click="onActionView('view-item', props.rowData)">
            <i class="zoom icon"></i>
          </button>
         
          <button class="ui basic button"
            @click="onAction('delete-item', props.rowData)">
            <i class="delete icon"></i>
          </button>
        </div>
     </div> 
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
import DetailRow from "./DetailRow";
import EventBus from './eventBus';
import CustomActions from './CustomActions';

// import LabelEdit from 'label-edit';

// Vue.component('custom-actions', CustomActions)
Vue.component('my-detail-row', DetailRow)

export default {
  components: {
    Vuetable,
    VuetablePagination,
    // CustomActions,
    // DetailRow
    // LabelEdit
  },
  data() {
    return {
      tbDomain:"",
      // refresh:"",
      api_path: API_PATH_MD_TABLE_ALL,
      perPage: 3,
      mdtables: [],
      allData: false,
      currentPage: 1,
      css: CssConfig,
      nodes:
        "MVR,Clue,Membership Profile,Policy Profile,ISO,Reserve,Claims Profile,Direct Report Network,Estimation,Survey,Online,Voice,Mail,Billing,Invoices,Payments,Campaign Management,Brand Promotions,Competitor Analysis",
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
        },
        {
          name:"__slot:actions",
          title: "Actions",
          callback: "displayDomain",
          titleClass: "center aligned"
        },
      ],
      sortOrder: [{ field: "id", direction: "desc" }]
    };
  },
  mounted() {
    //this.getMdTables();
       
    EventBus.$on('DIAGRAM_PUBLISHED', (diagramDomain) => {
      //pass domain from diagram to Table1
      this.diagram_to_tb(diagramDomain)
    })

  },
  created() {
      bus.$on("loadTable1", comp => {
      this.refreshtables(comp);
    }),
     bus.$on("nodeschecked", comp => {
      this.set_nodes_string(comp);
    }),
      bus.$on("loadTableData2", comp => {
        this.searchTable(comp);
      }),
      bus.$on("loadFirstTable", comp => {
        this.getTableDataByName(comp);
      });
  },

  methods: {
    displayDomain: function(value) {
      return value.join(",");
    },
    refreshtables(value) {
      this.api_path = API_PATH_MD_TABLE_ALL;
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
    },

    set_nodes_string: function(node_string) {
      this.nodes = "";
      for (var i = 0; i < node_string.length; i++) {
        if (node_string[i] != null) {
          if (node_string[i] == "2") {
            if (this.nodes == "") {
              this.nodes = "MVR";
            } else {
              this.nodes = this.nodes + "," + "MVR";
            }
          } else if (node_string[i] == "3") {
            if (this.nodes == "") {
              this.nodes = "Clue";
            } else {
              this.nodes = this.nodes + "," + "Clue";
            }
          } else if (node_string[i] == "4") {
            if (this.nodes == "") {
              this.nodes = "Membership Profile";
            } else {
              this.nodes = this.nodes + "," + "Membership Profile";
            }
          } else if (node_string[i] == "5") {
            if (this.nodes == "") {
              this.nodes = "Policy Profile";
            } else {
              this.nodes = this.nodes + "," + "Policy Profile";
            }
          } else if (node_string[i] == "6") {
            if (this.nodes == "") {
              this.nodes = "ISO";
            } else {
              this.nodes = this.nodes + "," + "ISO";
            }
          } else if (node_string[i] == "8") {
            if (this.nodes == "") {
              this.nodes = "Reserve";
            } else {
              this.nodes = this.nodes + "," + "Reserve";
            }
          } else if (node_string[i] == "9") {
            if (this.nodes == "") {
              this.nodes = "Claims Profile";
            } else {
              this.nodes = this.nodes + "," + "Claims Profile";
            }
          } else if (node_string[i] == "10") {
            if (this.nodes == "") {
              this.nodes = "Direct Report Network";
            } else {
              this.nodes = this.nodes + "," + "Direct Report Network";
            }
          } else if (node_string[i] == "11") {
            if (this.nodes == "") {
              this.nodes = "Estimation";
            } else {
              this.nodes = this.nodes + "," + "Estimation";
            }
          } else if (node_string[i] == "13") {
            if (this.nodes == "") {
              this.nodes = "Survey";
            } else {
              this.nodes = this.nodes + "," + "Survey";
            }
          } else if (node_string[i] == "14") {
            if (this.nodes == "") {
              this.nodes = "Online";
            } else {
              this.nodes = this.nodes + "," + "Online";
            }
          } else if (node_string[i] == "15") {
            if (this.nodes == "") {
              this.nodes = "Voice";
            } else {
              this.nodes = this.nodes + "," + "Voice";
            }
          } else if (node_string[i] == "16") {
            if (this.nodes == "") {
              this.nodes = "Mail";
            } else {
              this.nodes = this.nodes + "," + "Mail";
            }
          } else if (node_string[i] == "18") {
            if (this.nodes == "") {
              this.nodes = "Billing";
            } else {
              this.nodes = this.nodes + "," + "Billing";
            }
          } else if (node_string[i] == "19") {
            if (this.nodes == "") {
              this.nodes = "Invoices";
            } else {
              this.nodes = this.nodes + "," + "Invoices";
            }
          } else if (node_string[i] == "20") {
            if (this.nodes == "") {
              this.nodes = "Payments";
            } else {
              this.nodes = this.nodes + "," + "Payments";
            }
          } else if (node_string[i] == "22") {
            if (this.nodes == "") {
              this.nodes = "Campaign Management";
            } else {
              this.nodes = this.nodes + "," + "Campaign Management";
            }
          } else if (node_string[i] == "23") {
            if (this.nodes == "") {
              this.nodes = "Brand Promotions";
            } else {
              this.nodes = this.nodes + "," + "Brand Promotions";
            }
          } else if (node_string[i] == "24") {
            if (this.nodes == "") {
              this.nodes = "Competitor Analysis";
            } else {
              this.nodes = this.nodes + "," + "Competitor Analysis";
            }
          }
        }
      }
      console.log("LOOK AT ME!!!");
      console.log(this.nodes);
    },
    getTableDataByName(value) {
      console.log(value);
      this.api_path = API_PATH_MD_TABLE + "searchtable/" + value;
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
    },
    searchTable(value) {
      console.log(value);
      if (value == "") {
        this.api_path = API_PATH_MD_TABLE + "searchTBDiagramDomain/" + this.nodes;
      } else {
        console.log(this.nodes);
        this.api_path =
          API_PATH_MD_TABLE + "filterTableRecords/" + value + "/" + this.nodes;
        console.log(this.api_path);
      }

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
    
    // updateMdTable(row) {
    //   let uri = `http://localhost:8889/md_tables_indx/` + row.id;
    //   console.log("response edit");
    //   axios.post(uri, row).then(response => {
    //     console.log("response edit", response.status);
    //   });
    // },
    onActionEdit (row) {
      console.log('slot action: ' + action)
      this.api_path = API_PATH_MD_TABLE + row.id
      var self = this;
      this.$nextTick(() => {
        self.$refs.vuetable.refresh();
      });
      console.log('slot action: ' + action, data.name, index)
    },

    onActionView (data, field, event) {
      console.log('ViewClicked: ')
      this.$refs.vuetable.toggleDetailRow(data.id);
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
    },

   diagram_to_tb:function(diagramDomain){
     // get domain from diagram
     this.tbDomain = diagramDomain
     console.log("TB domain",this.tbDomain)

    //show diagram domains at table
    if(this.tbDomain != null){
       this.api_path = API_PATH_MD_TABLE + "searchTBDiagramDomain/" + this.tbDomain;
     }
    
     var self = this;
     this.$nextTick(() => {
        self.$refs.vuetable.refresh();
     });
    // },
     
    // updateTextEnter: function(row) {
    //   console.log(row);
    //   console.log(this.label);
    //   this.updateMdTable(row);
    //   row.comments = this.label;
    //   this.edit = false;
    }, 

      onLabelClick: function(row){
      for(var i = 0;i<this.vuetable.length;i++){
        if(row.id!=this.vuetable[i].id&&this.vuetable[i].edit){
          this.vuetable[i].edit = false;
          console.log(this.vuetable[i].id);
        }
      }
      this.label = row.texts;
      row.edit = true;
      console.log(row);
      } 
  }
};
</script>