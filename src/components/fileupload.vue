<template>
    <div :class="{ 'is-danger': fileSize > maxSize }">
        <input
            id="file-upload"
            ref="files"
            class="govuk-file-upload"
            name="file-upload"
            type="file"
            multiple
            @change="insertFile"
        />
        <button class="govuk-button" @click.prevent="addFiles()">
            Vložiť súbory
        </button>
        <div class="files-container">
            <table v-if="files.length > 0">
                <thead>
                    <tr>
                        <th>Názov súboru</th>
                        <th>Veľkosť</th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="(file, index) in files"
                        :key="index"
                        class="file"
                    >
                        <td>{{ file.name }}</td>
                        <td>{{ file.size / 1000 + " kB" }}</td>
                        <td>
                            <span @click.prevent="removeFile(index)"
                                >&#10060;</span
                            >
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        files: {
            type: Array,
            default: () => [],
        },
        maxSize: {
            type: Number,
            default: 10,
        },
    },
    data() {
        return {
            newFiles: this.files,
        };
    },
    computed: {
        fileSize: function() {
            let size = 0;
            this.newFiles.forEach((file) => {
                size = size + file.size;
            });
            //returns MiB
            return size / 1000 / 1000;
        },
    },
    methods: {
        addFiles: function() {
            this.$refs.files.click();
        },
        insertFile: function() {
            let uploadedFiles = this.$refs.files.files;

            for (var i = 0; i < uploadedFiles.length; i++) {
                if (
                    this.fileSize + uploadedFiles[i].size / 1000 / 1000 >
                    this.maxSize
                ) {
                    //TODO: Vypis chybovu hlasku
                } else {
                    this.files.push(uploadedFiles[i]);
                }
            }

            this.$emit("file", this.newFiles);
        },
        removeFile: function(index) {
            this.newFiles.splice(index, 1);
        },
    },
};
</script>

<style lang="scss" scoped>
input[type="file"] {
    display: none;
}
.file {
    span {
        cursor: pointer;
    }
}
//TODO: make global states/utils
.is-danger {
    border: 1px solid red;
}
</style>
