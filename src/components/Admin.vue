<template>
  <div class="adminArea">
    <el-row :gutter="30">
      <el-col :md="24" :lg="12">
        <div class="featureTitle">
          <i class="el-icon-edit"></i>&nbsp;新文章
        </div>
        <el-form :model="post">
          <el-form-item>
            <el-input v-model="post.title" placeholder="标题"></el-input>
          </el-form-item>
          <el-form-item>
            <el-input v-model="post.content" type="textarea" :rows="10" placeholder="内容"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="createPost()">提交</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :md="24" :lg="12">
        <div class="featureTitle">
          <i class="el-icon-s-tools"></i>&nbsp;博客情报更新
        </div>
        <el-card shadow="never">
          <el-form :model="blog" label-width="110px">
            <el-form-item label="博客名称">
              <el-input v-model="blog.blogName"></el-input>
            </el-form-item>
            <el-form-item label="副标题">
              <el-input v-model="blog.subtitle"></el-input>
            </el-form-item>
            <el-form-item label="管理员姓名">
              <el-input v-model="blog.adminName"></el-input>
            </el-form-item>
            <el-form-item label="管理员密码">
              <el-input v-model="blog.password" show-password placeholder="若不变更密码请留空"></el-input>
            </el-form-item>
            <el-form-item label="确认密码" v-if="blog.password">
              <el-input v-model="blog.confirm" show-password></el-input>
            </el-form-item>
            <el-form-item label="私有">
              <el-switch v-model="blog.private" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
            </el-form-item>
            <el-form-item label="阅览密码" v-if="blog.private">
              <el-input v-model="blog.viewPassword" show-password></el-input>
            </el-form-item>
            <el-form-item label="可否评论">
              <el-switch v-model="blog.commentable" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="updateBlogInfo()">更新</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
    <el-row>
      <div class="featureTitle">
        <i class="el-icon-document"></i>&nbsp;文章管理
      </div>
      <el-table :data="postList" border>
        <el-table-column label="操作" width="80px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="success"
              style="padding: 5px;"
              @click="editPost(scope.row.id)"
            >
              <i class="el-icon-edit"></i>
            </el-button>
            <el-button
              size="mini"
              type="danger"
              style="padding: 5px; margin-left: 5px"
              @click="deletePost(scope.row.id)"
            >
              <i class="el-icon-delete"></i>
            </el-button>
          </template>
        </el-table-column>
        <el-table-column label="标题" prop="title"></el-table-column>
      </el-table>
      <el-pagination
        small
        background
        layout="prev, pager, next"
        :total="blog.postsCount"
        :page-count="5"
        :current-page.sync="postCurrentPage"
        @current-change="changePostListPage"
      ></el-pagination>
    </el-row>
    <el-row>
      <div class="featureTitle">
        <i class="el-icon-chat-dot-square"></i>&nbsp;评论管理
      </div>
      <el-table :data="commentList" border>
        <el-table-column label="操作" width="50px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="danger"
              style="padding:5px"
              @click="deleteComment(scope.row.id)"
            >
              <i class="el-icon-delete"></i>
            </el-button>
          </template>
        </el-table-column>
        <el-table-column label="评论" prop="comment"></el-table-column>
      </el-table>
      <el-pagination
        small
        background
        layout="prev, pager, next"
        :total="blog.commentsCount"
        :page-count="5"
        :current-page.sync="commentCurrentPage"
        @current-change="changeCommentListPage"
      ></el-pagination>
    </el-row>
  </div>
</template>

<style scoped>
.el-card {
  color: #58666e;
  padding: 0 5px;
  font-size: 0.85rem;
}
.el-pagination {
  margin: 15px 0 10px;
  text-align: center;
}
div.adminArea {
  color: #58666e;
}
div.featureTitle {
  margin-bottom: 15px;
  margin-top: 20px;
}
</style>

<script>
import { get, post, msg } from "../utils/APIUtil";
import { URLs } from "../context/Consts";
import { mapActions } from "vuex";
import * as Types from "../store/types";

export default {
  methods: {
    createPost: function() {
      post(
        URLs.post.create,
        this.post,
        () => {
          this.$message.success("新文章已登录");
          this.post.title = "";
          this.post.content = "";
        },
        error => this.$message.error(msg(error))
      );
    },
    updateBlogInfo: function() {
      let blogInfo = this.blog;
      if (!this.blog.password) {
        blogInfo.password = "";
        blogInfo.confirm = "";
      }
      post(
        URLs.blog.update,
        blogInfo,
        response => {
          console.log(response.data);
          this.$message.success("更新博客情报成功");
          this.blog = response.data;
          this.SET_BLOG_INFO();
        },
        error => this.$message.error(msg(error))
      );
    },
    editPost: function(id) {
      this.$router.push({ name: "postEdit", params: { id } });
    },
    deletePost: function(id) {
      this.$confirm("是否删除此文章？", "确认", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          post(
            URLs.post.base + "/" + id + "/delete",
            null,
            () => {
              this.$message.success("文章删除成功");
              this.postCurrentPage = 1;
              this.getPostList(1, 10);
              this.getCommentList(1, 10);
              this.SET_BLOG_INFO();
            },
            error => this.$message.error(msg(error))
          );
        })
        .catch(() => {
          this.$message.info("删除已取消");
        });
    },
    getBlogInfo: function() {
      get(URLs.blog.base, null, response => (this.blog = response.data));
    },
    getPostList: function(pageNum, limit) {
      get(
        URLs.post.base,
        { pageNum, limit, isPostView: false },
        response => (this.postList = response.data)
      );
    },
    changePostListPage: function(pageNum) {
      this.getPostList(pageNum, 10);
    },
    getCommentList: function(pageNum, limit) {
      get(
        URLs.comment.base,
        { pageNum, limit },
        response => (this.commentList = response.data)
      );
    },
    changeCommentListPage: function(pageNum) {
      this.getCommentList(pageNum, 10);
    },
    deleteComment: function(id) {
      this.$confirm("是否删除此评论？", "确认", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          post(
            URLs.comment.base + "/" + id + "/delete",
            null,
            () => {
              this.$message.success("评论删除成功");
              this.commentCurrentPage = 1;
              this.getCommentList(1, 10);
              this.SET_BLOG_INFO();
            },
            error => this.$message.error(msg(error))
          );
        })
        .catch(() => {
          this.$message.info("删除已取消");
        });
    },
    ...mapActions([Types.SET_BLOG_INFO])
  },
  data() {
    return {
      post: {
        title: "",
        content: ""
      },
      blog: {},
      postList: [],
      commentList: [],
      postCurrentPage: 1,
      commentCurrentPage: 1
    };
  },
  mounted() {
    this.SET_BLOG_INFO();
    this.getBlogInfo();
    this.getPostList(1, 10);
    this.getCommentList(1, 10);
  }
};
</script>


