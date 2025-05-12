<template>
  <div class="uploadBox">
    <p class="upDesc">Click on the Open Upload Box to open the box modal</p>
    <button class="openUpload" @click="showOpenModal">Open upload Box</button>
    <div class="openParentModal" v-if="showUploadModal">
      <div class="upload-container">
        <div class="upload-header">
          <h2>Upload</h2>
          <button class="close-btn" @click="closeUploadModal">&times;</button>
        </div>
        <p class="upload-instructions">
          Add your documents here, and you can upload up to 5 files max
        </p>
        <div
          class="upload-area"
          @dragover.prevent="handleDragOver"
          @drop.prevent="handleDrop"
        >
          <div class="upload-icon">
            <img src="../assets/vector.png" alt="Upload" />
          </div>
          <p class="upload-prompt">Drag your file(s) to start uploading</p>
          <div class="orDiv">
            <span class="orLine"></span>
            <p class="or-text">OR</p>
          </div>

          <div class="upload-buttons">
            <label class="browse-btn">
              Browse files
              <input
                type="file"
                id="fileInput"
                ref="fileInput"
                class="hidden"
                multiple
                @change="handleFileInput"
                accept="application/pdf"
              />
            </label>
            <button class="link-btn" @click="toggleLinkInput">
              Enter website link
            </button>
          </div>
          <div v-if="showLinkInput" class="link-upload">
            <input
              v-model="linkInput"
              type="url"
              placeholder="Enter document link"
              class="link-field"
            />
            <button class="add-link-btn" @click="addLink">Add Link</button>
          </div>
        </div>
        <div class="uploaded-links">
          <div
            v-for="(link, index) in links"
            :key="'link-' + index"
            class="file-item"
          >
            <img
              src="../assets/link-icon.png"
              alt="Link Icon"
              class="file-icon"
            />
            <span class="file-name">{{ link }}</span>
            <button class="closePDf" @click="removeLink(index)">&times;</button>
          </div>
        </div>

        <p class="file-support-text">Only support .pdf</p>
        <div class="uploaded-files">
          <div
            v-for="(file, index) in uploadedFiles"
            :key="index"
            class="file-item"
          >
            <img
              src="../assets/pdf-icon.png"
              alt="PDF Icon"
              class="file-icon"
            />
            <span class="file-name">{{ file.name }}</span>
            <button class="closePDf" @click="removeFile(index)">&times;</button>
          </div>
        </div>

        <div class="email-input">
          <div class="email-icon">
            <span>&#9993;</span>
          </div>
          <input
            type="email"
            v-model="email"
            placeholder="Enter your email"
            class="email-field"
          />
        </div>
        <button class="generate-btn" @click="validateAndSubmit">
          Generate Schedule
        </button>
        <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
      </div>
      <div v-if="showSuccessModal" class="modal-overlay">
        <div class="modal-content">
          <div class="modal-icon">&#10003;</div>
          <p class="modal-text">Schedule generated successfully!</p>
          <button @click="closeModal" class="modal-close-btn">OK</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      uploadedFiles: [],
      email: "",
      errorMessage: "",
      showSuccessModal: false,
      showUploadModal: false,
      showLinkInput: false,
      linkInput: "",
      links: [],
    };
  },
  methods: {
    showOpenModal() {
      this.showUploadModal = true;
    },
    closeUploadModal() {
      this.showUploadModal = false;
    },
    toggleLinkInput() {
      this.showLinkInput = !this.showLinkInput;
      this.linkInput = "";
    },
    addLink() {
      const trimmed = this.linkInput.trim();
      if (!trimmed) {
        this.errorMessage = "Please enter a valid link.";
        return;
      }

      if (this.links.length >= 3) {
        this.errorMessage = "You can only add up to 3 links.";
        return;
      }

      this.links.push(trimmed);
      this.linkInput = "";
      this.showLinkInput = false;
      this.errorMessage = "";
    },

    closeModal() {
      this.showSuccessModal = false;
    },
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileInput(event) {
      const files = Array.from(event.target.files);
      this.validateFiles(files);
      event.target.value = "";
    },
    handleDragOver(event) {
      event.dataTransfer.dropEffect = "copy";
    },
    handleDrop(event) {
      const files = Array.from(event.dataTransfer.files);
      this.validateFiles(files);
    },
    removeLink(index) {
      this.links.splice(index, 1);
    },
    removeFile(index) {
      this.uploadedFiles.splice(index, 1);
      this.errorMessage = "";
    },
    addFiles(files) {
      const validFiles = files.filter(
        (file) => file.type === "application/pdf"
      );
      if (validFiles.length + this.uploadedFiles.length > 5) {
        this.errorMessage = "You can only upload up to 5 files.";
        return;
      }
      this.errorMessage = "";
      this.uploadedFiles = [...this.uploadedFiles, ...validFiles];
    },

    validateFiles(files) {
      const validFiles = files.filter(
        (file) => file.type === "application/pdf"
      );
      const totalFiles = this.uploadedFiles.length + validFiles.length;

      if (validFiles.length === 0) {
        this.errorMessage = "Only .pdf files are supported.";
        return;
      }

      if (totalFiles > 5) {
        const allowedCount = 5 - this.uploadedFiles.length;
        if (allowedCount > 0) {
          this.uploadedFiles = [
            ...this.uploadedFiles,
            ...validFiles.slice(0, allowedCount),
          ];
        }
        this.errorMessage = "You can only upload up to 5 PDF files.";
        return;
      }

      this.uploadedFiles = [...this.uploadedFiles, ...validFiles];
      this.errorMessage = "";
    },
    isValidEmail(email) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailRegex.test(email);
    },
    validateAndSubmit() {
      if (this.uploadedFiles.length === 0) {
        this.errorMessage = "No files uploaded. Please upload a .pdf file.";
        return;
      }
      if (!this.isValidEmail(this.email)) {
        this.errorMessage = "Please enter a valid email address.";
        return;
      }
      this.errorMessage = "";
      this.showSuccessModal = true;
      this.uploadedFiles = [];
      this.links = [];
      this.email = "";
      this.linkInput = "";
      this.showLinkInput = false;
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = "";
      }
    },
  },
};
</script>
