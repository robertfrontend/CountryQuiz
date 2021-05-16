<template>
  <div>
    <div class="modal-resultado">
      <img src="../assets/winner.svg" alt="" />
      <h1>Results</h1>
      <p>
        You got
        <span :class="correctas > 0 ? 'bien' : 'mal'">{{ correctas }}</span> correct
        answers
      </p>
      <button @click="resetear">Try again</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    correctas: Number,
  },
  methods: {
    resetear() {
      this.$emit("tryAain");
    },
  },
};
</script>

<style lang="css" scoped>
.modal-resultado {
  width: 350px;
  min-height: 300px;
  background: white;
  border-radius: 20px;
  padding: 1em 2em;
  text-align: center;
}
.modal-resultado h1 {
  padding-top: 1em;
  color: #1d355d;
  text-align: center;
  font-style: normal;
  font-weight: bold;
  font-size: 40px;
}
.modal-resultado p span {
  color: #6fcf97;
  font-size: 2em;
  font-weight: bold;
}
.bien {
  color: #6fcf97 !important;
}
.mal {
  color: red !important;
}
button {
  padding: 5px 2em;
  font-style: normal;
  font-weight: 600;
  font-size: 18px;
  line-height: 27px;
  border-radius: 8px;
  color: #1d355d;
  border: 1px solid #1d355d;
  cursor: pointer;
}
@media screen and (max-width: 550px) {
  .modal-resultado {
    width: 90%;
  }
}
</style>
