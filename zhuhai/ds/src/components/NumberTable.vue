<template>
  <div>
    <el-button size="mini" type="success" slot="reference" @click="dialogBatchFormVisible = true">批量证明</el-button>
  <el-table :data="numbers" @selection-change="handleSelectionChange">
    <el-table-column type="selection" width="55"></el-table-column>
    <el-table-column show-overflow-tooltip label="物品id" prop="pid" />
    <el-table-column show-overflow-tooltip label="承诺值1" prop="commit1" />
    <el-table-column show-overflow-tooltip label="承诺值2" prop="commit2" />
    <el-table-column show-overflow-tooltip label="范围" prop="range" />
    <el-table-column show-overflow-tooltip label="证明2" prop="proof" />
    <el-table-column show-overflow-tooltip label="证明1" prop="proofpre" />
    <el-table-column show-overflow-tooltip label="上传者" prop="uid" />
    <el-table-column show-overflow-tooltip label="上传时间" prop="timeStamp" width="250">
      <template slot-scope="scope">
        {{ scope.row.timeStamp }}
      </template>
    </el-table-column>
<!--    <el-table-column show-overflow-tooltip label="IP" prop="ip" width="130"></el-table-column>-->
<!--    <el-table-column show-overflow-tooltip label="加密策略" prop="policy" />-->
    <el-table-column show-overflow-tooltip label="标签" prop="tags">
      <template slot-scope="scope">
        <el-tag
          v-for="(tag, i) in filterEmpty(scope.row.tags)"
          :key="i"
          size="small"
          effect="plain"
        >
          {{ tag }}
        </el-tag>
      </template>
    </el-table-column>

    <el-table-column label="操作" align="right" width="200">
      <template slot-scope="scope">
        <el-button size="mini" type="success" @click="addProof(scope.row)">
          生成证明
        </el-button>
        <el-button size="mini" @click="verifyProof(scope.row)"> 验证证明 </el-button>
      </template>
    </el-table-column>
    <el-dialog title="生成证明" :visible.sync="dialogFormVisible" append-to-body>
      <el-form ref="createForm" :rules="applyRules" :model="form" label-width="80px">
        <el-form-item prop="range" label="范围">
          <el-input v-model="form.range"></el-input>
        </el-form-item>

        <!--          <el-form-item label="属性类型">-->
        <!--            <el-input-->
        <!--                placeholder="请输入用户或组织名"-->
        <!--                v-model="form.belongs"-->
        <!--                class="input-with-select"-->
        <!--            >-->
        <!--              <el-select v-model="form.role" slot="prepend" placeholder="请选择">-->
        <!--                <el-option label="用户属性" value="to"></el-option>-->
        <!--                <el-option label="组织属性" value="org"></el-option>-->
        <!--              </el-select>-->
        <!--            </el-input>-->
        <!--          </el-form-item>-->

        <!--          <el-form-item label="备注">-->
        <!--            <el-input type="textarea" v-model="form.remark"></el-input>-->
        <!--          </el-form-item>-->
      </el-form>
      <div slot="footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="createProof">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="生成批量证明" :visible.sync="dialogBatchFormVisible" append-to-body>
      <el-form ref="createForm" :rules="applyRules" :model="form" label-width="80px">
        <el-form-item prop="range" label="范围">
          <el-input v-model="form.range"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="dialogBatchFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="proofBatch">确 定</el-button>
      </div>
    </el-dialog>
  </el-table>
  </div>
</template>

<script>
// @ is an alias to /src
import { FileDownloader } from "@/mixins/Download";
import { FilterEmpty } from "@/mixins/FilterEmpty";
import { TimeFormat } from "@/mixins/TimeFormat";
import {numberApi} from "@/api/numbers";
import {Message} from "element-ui";
import {getters} from "@/store/store";

export default {
  name: "NumberTable",
  mixins: [FileDownloader, FilterEmpty, TimeFormat],

  data(){
    return{
      dialogFormVisible: false,
      dialogBatchFormVisible: false,
      multipleSelection: [],
      applyRules: {
        range: [{ required: true, trigger: "blur", message: "范围不能为空" }],
      },
    }
  },
  props: {
    numbers: {
      type: Array,
      default: undefined,
    },
    // dialogFormVisible: Boolean,
    form:{
      type: Object,
      default() {
        return {};
      }
    },
    // applyRules:{
    //   type: Object,
    // }
  },

  methods: {
    addProof(row) {
      this.dialogFormVisible = true
      this.form.pid = row.pid
      this.form.uid = row.uid
      this.form.timestamp = row.timeStamp
    },
    verifyProof(row) {
      const pid = row.pid
      const userName = row.uid
      const range = row.range
      const commit1 = row.commit1
      const commit2 = row.commit2
      const proof = row.proof
      const proofpre = row.proofpre
      console.log(proof)
      numberApi
          .verifyProof({userName, pid, range, commit1, commit2, proof, proofpre})
          .then((res) => {
            Message({
              message: "验证成功",
              duration: 5000,
              type: "success",
            });
            console.log(res);
          })
          .catch((e) => {
            Message({
              message: e.message,
              duration: 5000,
              type: "error",
            });
          });
    },
    createProof() {

      this.$refs.createForm.validate((valid) => {
        if (!valid) return;
        const pid = this.form.pid
        const userName = this.form.uid
        const range = this.form.range
        const timestamp = this.form.timestamp
        numberApi
            //生成证明
            .createProof({userName, pid, range, timestamp})
            .then((res) => {
              Message({
                message: "生成成功",
                duration: 5000,
                type: "success",
              });
              console.log(res);
            })
            .catch((e) => {
              Message({
                message: e.message,
                duration: 5000,
                type: "error",
              });
            });
      });
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },

    proofBatch() {
      this.$refs.createForm.validate((valid) => {
        if (!valid) return;
        const userName = getters.userName();
        const range = this.form.range
        let batch = this.multipleSelection.map((item) => {
          return Object.assign({}, {'pid': item.pid, "commit1": item.commit1, "timestamp":item.timeStamp})
        })
        let dtags = this.multipleSelection.map((item) => item.tags)
        const tags = [...new Set(dtags.flat())]
        console.log(tags)
        numberApi
            //生成证明
            .createProofBatch({userName, range, tags, batch})
            .then((res) => {
              Message({
                message: "生成成功",
                duration: 5000,
                type: "success",
              });
              console.log(res);
            })
            .catch((e) => {
              Message({
                message: e.message,
                duration: 5000,
                type: "error",
              });
            });
      });
    },
  }
};
</script>

<style scoped>
.el-tag {
  margin-right: 6px;
}
</style>