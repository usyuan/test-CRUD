<template>
  <div>
    <el-form
      :model="ruleForm"
      :rules="rules"
      status-icon
      ref="ruleForm"
      label-width="100px"
      class="demo-ruleForm"
    >
      <el-form-item label="姓名" prop="name">
        <el-input v-model="ruleForm.name" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="年龄" prop="age">
        <el-input type="number" v-model.number="ruleForm.age"></el-input>
      </el-form-item>
      <el-form-item>
        <!-- 送出按鈕 -->
        <el-button type="primary" size="mini" @click="setForm(submitBtnName)">{{
          submitBtnName == "add" ? "新增" : "更新"
        }}</el-button>
        <el-button size="mini" @click="resetForm">取消</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="resData"
      :key="tableKey"
      style="width: 100%"
      stripe
      class="data-table"
    >
      <el-table-column type="index" width="50"> </el-table-column>
      <el-table-column prop="name" label="姓名" width="180"> </el-table-column>
      <el-table-column prop="age" label="年齡"> </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" @click="editForm(scope.$index, scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            @click="deleteForm(scope.$index, scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      apiUrl: "https://test-crud.com.tw/testCRUD", //API路徑
      formName: "ruleForm",
      submitBtnName: "add", //送出按鈕文字顯示，add:新增、edit:更新
      resData: [],
      setResDataIndex: "", //編輯、刪除資料列的index值
      tableKey: 1, //解決element ui無法重新渲染問題
      ruleForm: { id: null, name: "", age: null }, //資料輸入
      rules: {
        name: [{ required: true, message: "請輸入姓名", trigger: "blur" }],
        age: [{ required: true, message: "請輸入年齡", trigger: "blur" }],
      },
      isBtnLock: false, //按鍵鎖
    };
  },
  created() {
    this.getForm();
  },
  methods: {
    // 取得資料
    getForm() {
      axios
        .get(this.apiUrl)
        .then((res) => {
          this.resData = res.data.result;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // 新增、編輯資料
    setForm(type) {
      //type，add:新增、edit:更新
      // 欄位驗證
      this.$refs[this.formName].validate((valid) => {
        if (valid && !this.isBtnLock) {
          // 資料驗證正確
          this.isBtnLock = true;
          let formData = new FormData();
          formData.append("name", this.ruleForm.name);
          formData.append("age", this.ruleForm.age);
          let data = {
            id: null,
            name: this.ruleForm.name,
            age: this.ruleForm.age,
          };

          if (type === "add") { //新增
            axios
              .post(this.apiUrl, data)
              .then((res) => {
                if (res.status === 200) {
                  this.resData.push(res.data.result);
                  this.$message({
                    showClose: true,
                    message: "新增成功",
                    type: "success",
                  });
                }
                this.isBtnLock = false;
              })
              .catch((error) => {
                console.log(error);
                this.isBtnLock = false;
              });
          } else { //編輯
            data.id = this.ruleForm.id;
            axios
              .put(this.apiUrl, data)
              .then((res) => {
                if (res.status === 200) {
                  let index = this.setResDataIndex;
                  this.resData[index] = res.data.result;
                  this.tableKey = !this.tableKey;
                  this.$message({
                    showClose: true,
                    message: "更新成功",
                    type: "success",
                  });
                }
                this.isBtnLock = false;
              })
              .catch((error) => {
                console.log(error);
                this.isBtnLock = false;
              });
          }
        } else {
          // 資料驗證錯誤
          return false;
        }
      });
    },
    // 選擇欲編輯的資料
    editForm(index, row) {
      console.log(index, row);
      this.submitBtnName = "edit";
      this.setResDataIndex = index;

      axios
        .get(this.apiUrl + row.id)
        .then((res) => {
          if (res.status === 200) {
            this.ruleForm = res.data.result[0];
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // 刪除資料
    deleteForm(index, row) {
      console.log(index, row);
      this.$confirm("是否確認刪除該筆資料？", "提示", {
        distinguishCancelAndClose: true,
        confirmButtonText: "確定",
        cancelButtonText: "取消",
      })
        .then(() => {
          if (!this.isBtnLock) {
            this.isBtnLock = true;
            this.setResDataIndex = index;

            axios
              .delete(this.apiUrl + row.id)
              .then((res) => {
                if (res.status === 200) {
                  let index = this.setResDataIndex;
                  this.resData.splice(index, 1);
                  this.$message({
                    showClose: true,
                    message: "刪除成功",
                    type: "success",
                  });
                }
                this.isBtnLock = false;
              })
              .catch((error) => {
                console.log(error);
                this.isBtnLock = false;
              });
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // 清空輸入值
    resetForm() {
      this.$refs[this.formName].resetFields();
      this.submitBtnName = "add";
    },
  },
};
</script>

<style lang="scss" scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.demo-ruleForm {
  width: 350px;
  margin: 30px auto 50px;
}
.data-table {
  border: 1px solid #c2c2c2;
  max-width: 600px;
  margin: 0 auto;
}
</style>
