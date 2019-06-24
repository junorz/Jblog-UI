<template>
  <div v-loading="loading">
    <i class="el-icon-edit"></i>&nbsp;编辑文章
    <el-form :model="post">
      <el-form-item>
        <el-input v-model="post.title" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="post.content" type="textarea" :rows="10" placeholder="内容"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="updatePost()">更新</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<style scoped>
.el-form {
  margin-top: 20px;
}
</style>

<script>
import { URLs } from "../context/Consts";
import { get, post, msg } from "../utils/APIUtil";

export default {
  data() {
    return {
      loading: true,
      post: {}
    };
  },
  methods: {
    updatePost: function() {
      post(
        URLs.post.base + "/" + this.$route.params.id + "/update",
        this.post,
        () => {
          this.$message.success("更新文章成功");
          this.$router.push({
            name: "posts",
            params: { id: this.$route.params.id }
          });
        },
        error => this.$message.error(msg(error))
      );
    }
  },
  mounted() {
    get(
      URLs.post.base + "/" + this.$route.params.id,
      null,
      response => {
        this.post = response.data;
        this.loading = false;
      },
      error => this.$message.error(msg(error))
    );
  }
};
</script>


