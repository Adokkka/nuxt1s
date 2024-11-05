<template>
  <v-container style="max-width: 100%">
    <v-row justify="center" align="center">
      <v-col sm="12" lg="12" md="12" style="margin-top: 20px">
        <v-row>
          <v-col cols="8"><h1>WebSRF</h1></v-col>
          <v-col cols="4" style="text-align: right">
            <v-btn class="primary" style="height: 48px" to="/create">
              WebSRF
              <v-icon right dark> mdi-plus </v-icon>
            </v-btn>
          </v-col>
        </v-row>

        <div>
          <v-data-table
            :headers="header"
            :items="this.ordersWithIndex"
            item-key="index"
            class="elevation-1 mt-10"
            style="cursor: pointer"
            :search="search"
            :footer-props="footerProps"
            :header-props="headerProps"
            show-expand
            :single-expand="true"
            @click:row="openForm"
          >
            <template #no-results>
              <v-alert :value="true" color="error" icon="warning">
                По вашему поиску "{{ search }}" не найдено результатов.
              </v-alert>
            </template>
          </v-data-table>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
// import format from 'date-fns/format';
// import ruLocale from 'date-fns/locale/ru';
export default {
  name: "list",
  data() {
    return {
      search: "",
      projects: [],
      loglist: [],
      headerProps: {
        sortByText: "Сортировать список",
      },
      footerProps: {
        "disable-items-per-page": true,
        "items-per-page-text": "Документов на странице",
      },
    };
  },
  head: {
    title: "WebSRF",
  },
  computed: {
    header() {
      return [
        { text: "id", value: "id" },
        {
          text: "Отправитель",
          value: "user.fio",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Номер проекта",
          value: "cost_name",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Наименование проекта",
          value: "cost_code",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Вид расходов",
          value: "type_expenditure",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Клиент",
          value: "client_name",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Дата создание",
          value: "created_at",
          format: "YYYY-MM-DD HH:mm:ss",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
        {
          text: "Статусы",
          value: "is_draft",
          cellClass: "cellCustom",
          class: "cellCustom",
        },
      ];
    },
    ordersWithIndex() {
      return this.loglist
        .sort((a, b) => b.id - a.id)
        .map((items, index) => ({
          ...items,
          index: index + 1,
        }));
    },
  },
  created() {
    console.log("info websrf info");
    this.initialize();
  },
  methods: {
    async initialize() {
      const response = await this.$axios.$get(
        `192.168.0.67/api/v1/websrf/all`,
        {}
      );
      console.log(response);

      for (let i = 0; i < response.length; i++) {
        response[i].created_at = moment(response[i].created_at).format(
          "DD MMMM yyyy hh:mm"
        );
        response[i].created_at.toLocaleString("en-US", {
          year: "numeric",
          month: "long",
          day: "numeric",
          hour: "2-digit",
          minute: "2-digit",
        });
        response[i].is_draft = response[i].is_draft
          ? "Отправлено на согласование"
          : "Черновик";
      }
      this.loglist = response;
      console.log("loglist", this.loglist);
      return this.loglist;
    },
    parseDate(date) {
      if (!date) return null;
      const [year, month, day] = date.split(/[.,\\/ -]/);
      // return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
      return `${day}/${month}/${year}`;
    },
    expandRow(item) {
      // would be
      // this.expanded = [item]
      // but if you click an expanded row, you want it to collapse
      this.expanded = item === this.expanded[0] ? [] : [item];
    },
    openForm(item) {
      // const queryParams = {
      //   item: item
      // }
      // if(item.doc_status !== 'Отправлено на согласование') {
      this.$router.push({
        path: "/create",
        query: {
          id: item.id,
        },
      });
      console.log(item.id);
    },
  },
};
</script>
