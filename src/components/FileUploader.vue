<style scoped lang="scss">
.file {
	width: 250px;
	display: inline-block;
	.file-name {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		margin-right: 15px;
		flex-grow: 1;
		align-self: center;
	}
	.preview {
		background-position: center;
		background-repeat: no-repeat;
		background-size: contain;
	}
}
</style>
<template>
	<div>
		<label v-if="label">{{ label }}</label>
		<form
			v-if="multiple || uploadedFiles.length === 0"
			enctype="multipart/form-data"
			novalidate
			class="mb-2">
			<div
				class="dropbox">
				<input
					type="file"
					:multiple="multiple"
					:name="uploadFieldName"
					:accept="accept"
					class="input-file"
					@change="filesChange($event.target.name, $event.target.files);">
				<p
					class="mb-0">
					Arrastra los archivos o haz click aquí
				</p>
			</div>
		</form>

		<div>
			<div
				v-for="(file, index) in uploadedFiles"
				:key="index"
				:class="{'mb-2 card': true, 'mr-2': true, 'file': true, 'w-100': preview}"
				:no-body="translated.length === 0">
				<div class="d-flex">
					<a
						class="file-name"
						:href="file.url"
						target="_blank">
						{{ file.original_filename }}
					</a>
					<button
						class="danger"
						@click="deleteUploadedFile(index)">
						<i
							class="bi bi-trash" />
					</button>
				</div>

				<div
					v-if="preview"
					class="preview"
					:style="{
						height: '100px',
						'background-image': 'url('+file.url+')',
					}" />
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'FileUploader',
    props: {
        value: [Object, Array],
        label: String,
        folder: {
            type: String,
            default: '',
        },
        multiple: {
            type: Boolean,
            default: false,
        },
        preview: {
            type: Boolean,
            default: false,
        },
        accept: {
            type: String,
            default: '*/*',
        },
    },
    data() {
        return {
            uploadedFiles: [],
            uploadFieldName: 'file',
        }
    },
    watch: {
        value() {
            this.init();
        },
    },
    created() {
        this.init();
    },
    methods: {
        init() {
            if (this.value) {
                this.uploadedFiles = this.multiple ? this.value : [this.value];
            }
        },
        upload(formData) {
            this.$api.post('files/upload', formData).then(x => {
                this.uploadedFiles.push(x.data.file);
                this.emitChange();
            }).catch(err => {
 console.error(err);
            });
        },
        filesChange(fieldName, fileList) {
            if (!fileList.length) return;

            Array.from(Array(fileList.length).keys()).map(file => {
                const formData = new FormData();
                formData.append(fieldName, fileList[file], fileList[file].name);
                formData.append('folder', this.folder)
                this.upload(formData);
                return file;
            });
        },
        deleteUploadedFile(index) {
            this.uploadedFiles.splice(index, 1);
            this.emitChange();
        },
        clear() {
            this.uploadedFiles = [];
        },
        emitChange() {
            this.$emit('input', this.multiple ? this.uploadedFiles : this.uploadedFiles[0]);
        },
    },
}
</script>

<style lang="scss">
.dropbox {
	outline: 2px dashed grey; /* the dash box */
	outline-offset: -10px;
	background: lightcyan;
	color: dimgray;
	padding: 10px 10px;
	min-height: 100px; /* minimum height */
	position: relative;
	cursor: pointer;
}

.input-file {
	opacity: 0; /* invisible but it's there! */
	width: 100%;
	height: 125px;
	position: absolute;
	cursor: pointer;
}

.dropbox:hover {
	background: lightblue; /* when mouse over to the drop zone, change color */
}

.dropbox p {
	font-size: 1.2em;
	text-align: center;
	padding: 25px 0;
}
</style>
