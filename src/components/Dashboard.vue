<template>
    <div id="dashboard">
        <section>
            <div class="col1">
                <div class="profile">
                    <h5>{{ userProfile.name }}</h5>
                    <p>{{ userProfile.title }}</p>
                    <div class="create-post">
                        <p>create a post</p>
                        <form @submit.prevent>
                            <textarea v-model.trim="post.content"></textarea>
                            <button @click="createPost" :disabled="post.content == ''" class="button">post</button>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col2">
                <transition name="fade">
                    <div v-if="hiddenPosts.length" @click="showNewPosts" class="hidden-posts">
                        <p>
                            Click to show <span class="new-posts">{{ hiddenPosts.length }}</span> 
                            new <span v-if="hiddenPosts.length > 1">posts</span><span v-else>post</span>
                        </p>
                    </div>
                </transition>
                <!-- comment modal -->
                <transition name="fade">
                    <div v-if="showCommentModal" class="c-modal">
                        <div class="c-container">
                            <a @click="closeCommentModal">X</a>
                            <p>add a comment</p>
                            <form @submit.prevent>
                                <textarea v-model.trim="comment.content"></textarea>
                                <button @click="addComment" :disabled="comment.content == ''" class="button">add comment</button>
                            </form>
                        </div>
                    </div>
                </transition>
                <div v-if="posts.length">
                    <div v-for="(post,idx) in posts" v-bind:key="idx" class="post">
                        <h5>{{post.userName}}</h5>
                        <span>{{ post.createdOn | formatDate }}</span>
                        <p>{{ post.content | trimLength }}</p>
                        <ul>
                            <li><a @click="openCommentModal(post)">comments {{ post.comments }}</a></li>
                            <li><a>likes {{ post.likes }}</a></li>
                            <li><a>view full post</a></li>
                        </ul>
                    </div>
                </div>
                <div v-else>
                    <p class="no-results">There are currently no posts</p>
                </div>
            </div>
        </section>
    </div>
</template>

<script>
    import moment from 'moment'
    import { mapState } from 'vuex';
    const fb = require('../firebaseConfig.js');
    
    export default {
        data() {
            return {
                post: {
                    content: '',
                },
                comment: {
                    postId: '',
                    userId: '',
                    content: '',
                    postComments: 0,
                },
                showCommentModal: false,
            }
        },
        computed: {
            ...mapState(['userProfile', 'currentUser', 'posts', 'hiddenPosts'])
        },
        methods: {
            createPost() {
                fb.postsCollection.add({
                    createdOn: new Date(),
                    content: this.post.content,
                    userId: this.currentUser.uid,
                    userName: this.userProfile.name,
                    comments: 0,
                    likes: 0,
                }).then(ref => {
                    this.post.content = '';
                }).catch(err => {
                    console.log(err);
                })
            },
            showNewPosts() {
                let updatedPostsArray = this.hiddenPosts.concat(this.posts);
                this.$store.commit('setHiddenPosts', null);
                this.$store.commit('setPosts', updatedPostsArray);
            },
            openCommentModal(post) {
                this.comment.postId = post.id
                this.comment.userId = post.userId
                this.comment.postComments = post.comments
                this.showCommentModal = true
            },
            closeCommentModal() {
                this.comment.postId = ''
                this.comment.userId = ''
                this.comment.content = ''
                this.showCommentModal = false
            },
            addComment() {
                let postId = this.comment.postId
                let postComments = this.comment.postComments
                
                fb.commentsCollection.add({
                    createdOn: new Date(),
                    content: this.comment.content,
                    postId: postId,
                    userId: this.currentUser.uid,
                    userName: this.userProfile.name
                }).then(doc => {
                    fb.postsCollection.doc(postId).update({
                        comments: postComments + 1
                    }).then(() => {
                        this.closeCommentModal()
                    })
                }).catch(err => {
                    console.log(err)
                })
            }
        },
        filters: {
            formatDate(val) {
                if(!val) return '-';

                let date = val.toDate();
                return moment(date).fromNow();
            },
            trimLength(val) {
                if (val.length < 200) return val;
                return `${val.substring(0,200)}...`;
            },
        },
    }
</script>