<template>
  <Card title="我的数额">
    <NumberTable :numbers="numbers" />
  </Card>
</template>

<script>
// @ is an alias to /src
import Card from "@/components/Card.vue";
import { getters } from "@/store/store";
// import { FileDownloader } from "@/mixins/Download";
// import { FilterEmpty } from "@/mixins/FilterEmpty";
import NumberTable from "@/components/NumberTable";
import {numberApi} from "@/api/numbers";

export default {
  name: "Mine",
  // mixins: [FileDownloader, FilterEmpty],
  components: {
    NumberTable,
    Card,
  },
  data() {
    return {
      numbers: [],
      bookmark: "",
      // dialogFormVisible: false,
      form: {
          pid: "",
          commit: "",
          uid: "",
          range: "",
      },

    };
  },

  mounted() {
    const tag = "";
    const userName = getters.userName();
    const { bookmark } = this;

    numberApi.numbers({ userName, tag, bookmark })
      .then((_) => {
        this.numbers = _.bulletProofs;
        this.bookmark = _.bookmark;
      })
      .catch(console.log);
  },
};
</script>

<style scoped>
.el-tag {
  margin-right: 6px;
}
</style>