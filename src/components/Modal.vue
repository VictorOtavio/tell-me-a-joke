<template>
  <div v-show="show" class="modal-container">
    <div class="modal">
      <button
        class="close"
        title="Close"
        @click.prevent="closeModal"
        v-show="$store.state.faceMood === 'happy' || error"
        v-text="'X'"
      />
      <LoadingSpinner v-if="isLoading" />
      <Quote v-else :text="joke" />
    </div>
  </div>
</template>

<script>
import time2read from "@/utils/time2read";
import Quote from "@/components/Quote";
import JokesAPI from "@/services/JokesAPI";
import LoadingSpinner from "@/components/LoadingSpinner";

export default {
  name: "Modal",

  components: {
    Quote,
    LoadingSpinner
  },

  props: {
    // Indicates if the modal window is open
    show: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      isLoading: true,
      joke: "",
      error: false
    };
  },

  watch: {
    // Search for a joke whenever the modal window is opened.
    show(val) {
      if (val) {
        this.getJoke();
      }
    }
  },

  methods: {
    /**
     * Searches for a joke through the API and, if a joke
     * is found, simulates its reading.
     */
    async getJoke() {
      this.isLoading = true;

      const response = await JokesAPI.getJoke();
      this.joke = response.text;

      this.isLoading = false;

      if (response.success) {
        this.readJoke();
      } else {
        this.error = true;
      }
    },

    /**
     * Simulates text reading time.
     */
    readJoke() {
      // Calculate time to read the joke
      const milisecs = time2read(this.joke);

      // Reading the joke
      setTimeout(() => {
        this.$store.dispatch("neutralFace");
        setTimeout(() => {
          this.$store.dispatch("happyFace");
        }, milisecs);
      }, milisecs);
    },

    /**
     * Closes the modal window.
     */
    closeModal() {
      this.$emit("update:show", false);
      this.$store.dispatch("neutralFace");
    }
  }
};
</script>

<style lang="scss" scoped>
.modal-container {
  overflow: hidden;
  position: absolute;
  width: 100vw;
  height: 100vh;
}

.modal {
  position: relative;
  background-color: #fffdf5;
  color: #3f3f5a;
  border-radius: 6px;
  max-width: 550px;
  max-height: calc(100vh - 80px);
  overflow: hidden;
  box-sizing: border-box;
  padding: 20px 50px;
  margin: 40px;
}

.close {
  border: none;
  cursor: pointer;
  background-color: #f1efe6;
  box-sizing: border-box;
  color: rgb(0, 0, 0);
  font-weight: 700;
  width: 25px;
  height: 25px;
  margin: 0;
  padding: 1px 6px;
  outline: none;
  position: absolute;
  right: 5px;
  top: 5px;
  text-align: center;
  border-radius: 50%;

  &:focus,
  &:hover {
    background-color: #d1cfc6;
  }
}

@media (max-width: 768px) {
  .modal-container {
    position: relative;
    height: auto;
  }

  .modal {
    margin: 20px;
  }
}
</style>
