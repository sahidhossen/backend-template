<template>
    <div>
    <media-modal v-if="$store.state.showUploadMediaModal" :className="'joomla-upload-modal'" :size="'sm'" @close="close()" :showClose="false" label-element="uploadMediaTitle">
        <div slot="body">
            <div 
                class="joomla-media-upload-drag-container"
                @dragenter="onDragEnter"
                @drop="onDrop"
                @dragover="onDragOver"
                @dragleave="onDragLeave"
                >
                <div class="joomla-recent-uploaded-media">
                    <div class="joomla-recent-uploaded-media-item" v-for="image in uploadedItems">
                        <div class="joomla-media-loader-box" v-if="isLoading">
                            <div class="media-loader" v-if="isLoading"></div>
                        </div>
                        <img :src="image.src" />
                    </div>
                </div>
                <button class="btn btn-success" @click="chooseFiles"> Upload File </button>
            </div>
            
        </div>
        <div slot="footer">
            <button class="btn btn-secondary" @click="close()">{{ translate('JCANCEL') }}</button>
        </div>
    </media-modal>
    
      <input type="file" class="hidden"
           :name="name"
           :multiple="multiple"
           :accept="accept"
           @change="upload_file"
           ref="fileInput">
    </div>
</template>
<script>
    import * as types from "./../../store/mutation-types";
    export default {
        name: 'media-upload',
        props: {
            accept: {
                type: String,
            },
            extensions: {
                default: () => [],
            },
            name: {
                type: String,
                default: 'file',
            },
            multiple: {
                type: Boolean,
                default: true,
            },
            images: {
                type: Array,
                default: () => { return [] }
            }
        },
        computed: {
            isLoading() {
                return this.$store.state.isLoading;
            },
            uploadedItems(){
                return this.$store.state.lastUploadedFile;
            }
        },
        methods: {
            /* Close modal */
            close() {
                this.$store.commit(types.HIDE_UPLOAD_MEDIA_MODAL);
            },
            /* Open the choose-file dialog */
            chooseFiles(e) {
                this.$refs['fileInput'].click();
            },
            /* Upload files */
            upload_file(e) {
                e.preventDefault();
                const files = e.target.files;
                // Loop through array of files and upload each file
                for (let file of files) {
                    this.upload(file);
                }
            },
             // Listeners for drag and drop
            // Fix for Chrome
            onDragEnter(e) {
                e.stopPropagation();
                return false;
            },


            // Notify user when file is over the drop area
            onDragOver(e) {
                e.preventDefault();
                document.querySelector('.joomla-media-upload-drag-container').classList.add('active');
                return false;
            },

            /* Upload files */
            upload(file) {
                // Create a new file reader instance
                let reader = new FileReader();

                // Add the on load callback
                reader.onload = (progressEvent) => {
                    const result = progressEvent.target.result,
                        splitIndex = result.indexOf('base64') + 7,
                        content = result.slice(splitIndex, result.length);
                    // Upload the file
                    this.$store.dispatch('uploadFile', {
                        name: file.name,
                        parent: this.$store.state.selectedDirectory,
                        content: content,
                    });
                    this.$store.commit(types.SET_LAST_UPLOADED_FILES, {src:result,name: file.name})
                };

                reader.readAsDataURL(file);
            },

            // Logic for the dropped file
            onDrop(e) {
                e.preventDefault();

                // Loop through array of files and upload each file
                if (e.dataTransfer && e.dataTransfer.files && e.dataTransfer.files.length > 0) {
                    for (let i = 0, f; f = e.dataTransfer.files[i]; i++) {
                        document.querySelector('.joomla-media-upload-drag-container').classList.remove('active');
                        this.upload(f);
                    }
                }
                document.querySelector('.joomla-media-upload-drag-container').classList.remove('active');
            },

            // Reset the drop area border
            onDragLeave(e) {
                e.stopPropagation();
                e.preventDefault();
                document.querySelector('.joomla-media-upload-drag-container').classList.remove('active');
                return false;
            },
        },
        mounted(){
            // this.$store.commit(types.SET_LAST_UPLOADED_FILES, [])
        },
        created() {
            // Listen to the toolbar upload click event
            // MediaManager.Event.listen('onClickUpload', () => this.chooseFiles());
            MediaManager.Event.listen('onClickUpload', () => this.$store.commit(types.SHOW_UPLOAD_MEDIA_MODAL));
        },
    }
</script>