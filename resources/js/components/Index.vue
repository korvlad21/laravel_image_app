<template>
    <div class="w-25">
        <input v-model="title" type="text" class="mb-3 form-control" placeholder="title">
        <div ref="dropzone" class="mb-3 btn d-block p-5 bg-dark text-center text-light">
            Upload
        </div>
        <div class="mb-3">
            <vue-editor useCustomImageHandler @image-added="handleImageAdded" v-model="content" />
        </div>
        <input @click.prevent="store" type="submit" class="btn btn-primary" value="add">

        <div class="mt-5">
            <div v-if="post">
                <h4>{{post.title}}</h4>
                <div v-for="image in post.images" class="mb-3">
                    <img :src="image.preview_url" class="mb-3">
                    <img :src="image.url">
                </div>
                <div class="ql-editor" v-html="post.content"></div>
            </div>
        </div>
    </div>
</template>

<script>
import Dropzone from 'dropzone'
import { VueEditor } from "vue2-editor";

export default {
    

    name: "Index",
    data(){
        return{
            dropzone: null,
            title: null,
            post: null,
            content: null
        }
    },

    components: { 
        VueEditor 
    },

    mounted() {
        this.getPosts()
        this.dropzone = new Dropzone(this.$refs.dropzone, {
            url: '/api/posts',
            autoProcessQueue: true,
            addRemoveLinks: true

        })
        
    },

    methods:{
        store(){
            const data = new FormData()
            const files = this.dropzone.getAcceptedFiles()
            files.forEach(file => {
                data.append('images[]', file)
                this.dropzone.removeFile(file)
            })
            data.append('title', this.title)
            data.append('content', this.content)
            this.title = '';
            this.content = '';
            axios.post('/api/posts', data)
            .then(res => {
                this.getPosts()
            })
        },

        getPosts() {
            axios.get('/api/posts')
            .then(res => {
                this.post = res.data.data
            })
        },
        handleImageAdded(file, Editor, cursorLocation, resetUploader){
            const formData = new FormData();
            formData.append("image", file);

            axios.post("/api/posts/images",formData)
                .then(result => {
                    const url = result.data.url; // Get url from response
                    Editor.insertEmbed(cursorLocation, "image", url);
                    resetUploader();
                })
                .catch(err => {
                console.log(err);
                });
            }
    }
}

</script>

<style>
.dz-success-mark,
.dz-error-mark{
    display: none;
}
</style>