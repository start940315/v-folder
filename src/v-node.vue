<template>
  <li :class="isChosen" :key="data.level">
    <i
      class="fa"
      :class="folderClass"
      @click="notify('unfold')"
    ></i>
    <span>
      <i class="fa" :class="checkboxClass" @click="notify('change')"></i>
      <span @click="notify('choose')" :title="data.name">{{data.name}}</span>
    </span>
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
    name: 'v-node',
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
      folderClass() {
        let data = this.data;
        let folderLoding = data.status === 'loading';
        let folderOpen = data.canOpen && data.open;
        let isEmpty = !data.canOpen && data.status === 'done';
        return {
          'fa-spinner cursor-progress': folderLoding,
          'fa-folder-open-o': !folderLoding && folderOpen,
          'fa-folder-o': !folderLoding && !folderOpen,
          'cursor-no-ops': isEmpty
        };
      },
      checkboxClass() {
        return classNames[this.data.check + 1];
      },
      isChosen() {
        return (this.data.chosen ? "chosen " : "") + "v-node";
      }
    }
  };
</script>
