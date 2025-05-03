<template>
  <div class="carrossel">
    <!-- Área do slider que mostra as imagens -->
    <div
      class="slider"
      :style="{ transform: `translateX(-${currentIndex * 100}%)` }"
    >
      <div class="slide" v-for="(image, index) in images" :key="index">
        <img :src="image" :alt="`Pet ${index + 1}`" />
      </div>
    </div>

    <!-- Notificação -->
    <div v-if="notification" :class="['notification', notification.type]">
      {{ notification.message }}
    </div>

    <!-- Controles na parte inferior -->
    <div class="controls">
      <div class="botoes">
        <!-- Botão Anterior -->
        <button @click="prevSlide">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <path
              d="M15 18L9 12L15 6"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </button>

        <!-- Botão Adicionar -->
        <button class="add-btn" @click="addImage">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <path
              d="M12 5V19M5 12H19"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </button>

        <!-- Botão Excluir -->
        <button
          class="delete-btn"
          @click="deleteCurrentImage"
          v-if="images.length > 1"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <path
              d="M18 6L6 18M6 6L18 18"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </button>

        <!-- Botão Salvar -->
        <button
          class="save-btn"
          @click="saveImages"
          v-if="images.length > 0"
          :disabled="isSaving"
        >
          <span v-if="isSaving">Salvando...</span>
          <span v-else>
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
              <path
                d="M19 21H5C3.89543 21 3 20.1046 3 19V5C3 3.89543 3.89543 3 5 3H16.1716C16.702 3 17.2107 3.21071 17.5858 3.58579L20.4142 6.41421C20.7893 6.78929 21 7.29799 21 7.82843V19C21 20.1046 20.1046 21 19 21Z"
                stroke="currentColor"
                stroke-width="2"
              />
              <path d="M17 21V13H7V21" stroke="currentColor" stroke-width="2" />
              <path
                d="M10 7H14"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
          </span>
        </button>

        <!-- Botão Próximo -->
        <button @click="nextSlide">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <path
              d="M9 18L15 12L9 6"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </button>
      </div>

      <!-- Indicadores de posição -->
      <div class="indicators">
        <span
          v-for="(_, index) in images"
          :key="'indicator-' + index"
          :class="{ active: currentIndex === index }"
          @click="currentIndex = index"
        ></span>
      </div>
    </div>

    <!-- Input file oculto para selecionar imagens -->
    <input
      type="file"
      ref="fileInput"
      @change="handleFileUpload"
      accept="image/*"
      multiple
      style="display: none"
    />
  </div>
</template>

<script>
const defaultImages = [
  new URL("../assets/cat1.png", import.meta.url).href,
  new URL("../assets/cat3.png", import.meta.url).href,
  new URL("../assets/cat6.png", import.meta.url).href,
  new URL("../assets/dog4.png", import.meta.url).href,
  new URL("../assets/dog5.png", import.meta.url).href,
  new URL("../assets/dog402.png", import.meta.url).href,
];

export default {
  name: "Carrossel",
  data() {
    return {
      currentIndex: 0,
      images: [...defaultImages],
      isSaving: false,
      notification: null,
    };
  },
  methods: {
    prevSlide() {
      this.currentIndex =
        (this.currentIndex - 1 + this.images.length) % this.images.length;
    },
    nextSlide() {
      this.currentIndex = (this.currentIndex + 1) % this.images.length;
    },
    addImage() {
      this.$refs.fileInput.click();
    },

    async compressImage(imageSrc) {
      return new Promise((resolve) => {
        const img = new Image();
        img.src = imageSrc;

        img.onload = () => {
          const canvas = document.createElement("canvas");
          const ctx = canvas.getContext("2d");

          // Definir tamanho máximo (800x600)
          const MAX_WIDTH = 800;
          const MAX_HEIGHT = 600;
          let width = img.width;
          let height = img.height;

          // Redimensionar se necessário
          if (width > height) {
            if (width > MAX_WIDTH) {
              height *= MAX_WIDTH / width;
              width = MAX_WIDTH;
            }
          } else {
            if (height > MAX_HEIGHT) {
              width *= MAX_HEIGHT / height;
              height = MAX_HEIGHT;
            }
          }

          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(img, 0, 0, width, height);

          // Converter para JPEG com qualidade 70%
          resolve(canvas.toDataURL("image/jpeg", 0.7));
        };

        img.onerror = () => resolve(imageSrc); // Se der erro
      });
    },

    async saveImages() {
      try {
        this.isSaving = true;

        //  15 imagens (ajustável)
        const imagesToSave = this.images.slice(0, 15);

        //  Comprimir as imagens antes de salvar
        const compressedImages = await Promise.all(
          imagesToSave.map((img) => this.compressImage(img))
        );

      
        localStorage.setItem(
          "savedPetImages",
          JSON.stringify(compressedImages)
        );

        this.showNotification("Imagens salvas com sucesso!", "success");
      } catch (e) {
        console.error("Erro ao salvar imagens:", e);
        this.showNotification(
          "Erro ao salvar imagens! O limite de armazenamento foi excedido.",
          "error"
        );
      } finally {
        this.isSaving = false;
      }
    },

    async handleFileUpload(event) {
      const files = event.target.files;
      if (!files || files.length === 0) return;

      try {
        const MAX_FILE_SIZE = 15 * 1024 * 1024; // 15MB
        const VALID_MIME_TYPES = [
          "image/jpeg",
          "image/png",
          "image/gif",
          "image/webp",
          "image/svg+xml",
          "image/bmp",
          "image/tiff",
          "image/avif",
          "image/*", // Aceita qualquer tipo de imagem
        ];

        for (const file of files) {
          if (
            !VALID_MIME_TYPES.some((type) => {
              return (
                type === file.type ||
                (type === "image/*" && file.type.startsWith("image/"))
              );
            })
          ) {
            this.showNotification(`Tipo não suportado: ${file.name}`, "error");
            continue;
          }

          if (file.size > MAX_FILE_SIZE) {
            this.showNotification(
              `Arquivo muito grande (${(file.size / (1024 * 1024)).toFixed(
                2
              )}MB): ${file.name}`,
              "error"
            );
            continue;
          }

          await this.processImageFile(file);
        }
      } catch (error) {
        console.error("Erro ao processar imagens:", error);
        this.showNotification("Erro ao carregar imagens!", "error");
      } finally {
        event.target.value = "";
      }
    },

    async processImageFile(file) {
      return new Promise((resolve) => {
        const reader = new FileReader();

        reader.onload = (e) => {
          this.images.push(e.target.result);
          this.currentIndex = this.images.length - 1;
          resolve();
        };

        reader.onerror = () => {
          this.showNotification(`Erro ao ler o arquivo: ${file.name}`, "error");
          resolve();
        };

        reader.readAsDataURL(file);
      });
    },

    deleteCurrentImage() {
      this.images.splice(this.currentIndex, 1);
      if (this.currentIndex >= this.images.length) {
        this.currentIndex = Math.max(0, this.images.length - 1);
      }
      this.showNotification("Imagem removida!", "success");
    },

    showNotification(message, type = "success") {
      this.notification = { message, type };
      setTimeout(() => (this.notification = null), 3000);
    },
  },

  mounted() {
    const savedImages = localStorage.getItem("savedPetImages");
    if (savedImages) {
      try {
        this.images = JSON.parse(savedImages);
      } catch (e) {
        console.error("Erro ao carregar imagens salvas:", e);
      }
    }
  },
};
</script>

<style scoped>
.carrossel {
  position: relative;
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
  aspect-ratio: 16/9;
  overflow: hidden;
  border-radius: 20px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
  background: white;
}

.slider {
  display: flex;
  height: 100%;
  transition: transform 0.6s ease-in-out;
}

.slide {
  min-width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.slide img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  transition: transform 0.5s ease;
}

.slide img:hover {
  transform: scale(1.03);
}

.controls {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 15px 0;
  background: rgba(255, 255, 255, 0.9);
  border-bottom-left-radius: 20px;
  border-bottom-right-radius: 20px;
}

.botoes {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 10px;
}

.botoes button {
  padding: 10px;
  border: none;
  border-radius: 50%;
  color: white;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
}

.add-btn {
  background: #4caf50;
}
.add-btn:hover {
  background: #3d8b40;
}

.delete-btn {
  background: #f44336;
}
.delete-btn:hover {
  background: #d32f2f;
}

.save-btn {
  background: #ff9800;
}
.save-btn:hover {
  background: #e68a00;
}

.botoes button:not(.add-btn):not(.save-btn):not(.delete-btn) {
  background: #4a6fa5;
}
.botoes button:not(.add-btn):not(.save-btn):not(.delete-btn):hover {
  background: #3a5a8f;
}

.save-btn[disabled] {
  opacity: 0.7;
  cursor: not-allowed;
}

.indicators {
  display: flex;
  justify-content: center;
  gap: 8px;
}

.indicators span {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #ccc;
  cursor: pointer;
  transition: background 0.3s;
}

.indicators span.active {
  background: #4a6fa5;
  transform: scale(1.2);
}

.notification {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 12px 24px;
  border-radius: 8px;
  color: white;
  animation: slideIn 0.3s ease-out;
  z-index: 1000;
  font-size: 14px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.notification.success {
  background-color: #4caf50;
}

.notification.error {
  background-color: #f44336;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .carrossel {
    width: 95%;
    aspect-ratio: 1/1;
  }

  .botoes button {
    width: 36px;
    height: 36px;
    padding: 8px;
  }
}

@media (max-width: 480px) {
  .carrossel {
    aspect-ratio: 9/16;
  }

  .botoes {
    gap: 6px;
  }

  .botoes button {
    width: 32px;
    height: 32px;
  }

  .notification {
    padding: 10px 20px;
    font-size: 12px;
  }
}
</style>
