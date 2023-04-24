<template>
  <Card title="证明搜索">
    <div style="margin-bottom: 15px">
      <el-row :gutter="10">
        <el-col :span="7">
          <el-input placeholder="请输入要搜索的用户id" v-model="filter.userName"></el-input>
        </el-col>
        <el-col :span="7">
        <el-input placeholder="请输入要搜索的物品id" v-model="filter.pid"></el-input>
        </el-col>
        <el-col :span="7">
        <el-input placeholder="请输入要搜索的tag" v-model="filter.tag"></el-input>
        </el-col>
        <el-col :span="2">
          <el-button style="float: right" type="primary" @click="search">筛选</el-button>
        </el-col>
      </el-row>
    </div>
    <NumberTable :numbers="numbers" />
  </Card>
</template>

<script>
// @ is an alias to /src
import Card from "@/components/Card.vue";
import { TimeFormat } from "@/mixins/TimeFormat";
import NumberTable from "@/components/NumberTable";
import {numberApi} from "@/api/numbers";

export default {
  name: "Search",
  mixins: [TimeFormat],
  components: {
    NumberTable,
    Card,
  },

  data() {
    return {
      filter: {},
      numbers: [],
      bookmark: "",
    };
  },

  methods: {
    search() {
      this.bookmark = "";
      const { userName, pid, tag } = this.filter;
      const { bookmark } = this;

      numberApi.numbers({ userName, pid, tag, bookmark })
          .then((_) => {
            this.numbers = _.bulletProofs;
            this.bookmark = _.bookmark;
          })
          .catch(console.log);
    },
  },
};
</script>
