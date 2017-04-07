<template>
  <li class="v-leaf" :key="data.level">
    <i class="fa" :class="className" @click="notify('change')"></i>
    <span :class="isChosen" @click="notify('choose')">{{data.name}}</span>
  </li>
</template>

<script>
  import EventMixin from './mixin';

  const classNames = [
    'fa-square-o',
    'fa-minus-square-o',
    'fa-check-square-o',
  ];

  export default {
    name: 'v-leaf',
    mixins: [EventMixin],
    props: {
      data: {
        type: Object,
        required: true
      },
      uid: {
        type: [String, Number],
        required: true
      }
    },
    computed: {
      className() {
        return classNames[this.data.check + 1];
      },
      isChosen() {
        return this.data.chosen ? "chosen" : "";
      }
    }
  };
</script>
