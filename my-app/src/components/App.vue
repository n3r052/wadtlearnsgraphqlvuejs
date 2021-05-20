<template>
  <div id="app">
    <!-- <p class="username">{{ currentUser.username }}'s posts:</p>
    <ul>
      <li v-for="post in posts" :key="post.id">{{ post.content }}</li>
    </ul>
    <div class="text-gray-500">
      <input v-model="newPostContent" />
      <button @click="addPost()">Add Post</button>
    </div> -->
    <table v-for="post in posts" :key="post.id" width="100%">
      <tr>
        <th>NAME</th>
        <th>TITLE</th>
        <th>USER ID</th>
        <th>ID</th>
      </tr>

      <tr>
        <td>
          <img src="../assets/img/user.png" width= "30px" height = "30px" alt="">
          {{ currentUser.username }}</td>
        <td>{{ post.content }}</td>
        <td>{{ post.userId }}</td>
        <td>{{ post.id }}</td>
      </tr>

    </table>
  </div>

</template>

<style scoped>
table, tr, th, td{
  border: 1px solid black;
  border-collapse: collapse;
  padding: 5px;
}

td{
  text-align: center;
}

</style>

<script>
import gql from "graphql-tag";

const CURRENT_USER = gql`
  query {
    currentUser {
      id
      username
    }
  }
`;

const POSTS_BY_USER = gql`
  query($userId: String!) {
    postsByUser(userId: $userId) {
      id
      content
    }
  }
`;

const ADD_POST = gql`
  mutation($content: String!) {
    addPost(content: $content) {
      id
      content
    }
  }
`;

function updateAddPost(cache, result) {
  let newPost = result.data.addPost;

  console.log(newPost.id);

  let cacheId = {
    query: POSTS_BY_USER,
    variables: { userId: this.currentUser.id }
  };

  const data = cache.readQuery(cacheId);
  const newData = [...data.postsByUser, newPost];

  cache.writeQuery({
    ...cacheId,
    data: { postsByUser: newData }
  });
}

export default {
  name: "app",
  data: function() {
    return {
      currentUser: { username: "user" },
      posts: [],
      newPostContent: ""
    };
  },

  methods: {
    addPost() {
      this.$apollo.mutate({
        mutation: ADD_POST,
        variables: { content: this.newPostContent },
        update: updateAddPost.bind(this),

        // NEW
        optimisticResponse: {
          __typename: "Mutation",
          addPost: {
            __typename: "Post",
            id: "xyz-?",
            content: this.newPostContent,
            userId: this.currentUser.id
          }
        }
      });

      this.newPostContent = "";
    }
  },

  apollo: {
    currentUser: CURRENT_USER,
    posts: {
      query: POSTS_BY_USER,
      variables() {
        return { userId: String(this.currentUser.id) };
      },
      update(data) {
        return data.postsByUser;
      }
    }
  }
};
</script>


