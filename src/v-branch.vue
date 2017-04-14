<template>
  <li :key="data.node.level" class="v-branch">
    <ul class="v-branch-body">
      <v-node :canChosen="canChosen" :data="data.node" :uid="uid"></v-node>
      <v-branch
        v-show="data.node.open"
        v-for="branch in data.branches"
        :key="data.node.name"
        :data="branch"
        :uid="uid"
        :canChosen="canChosen"
      ></v-branch>
      <v-leaf
        v-show="data.node.open"
        v-for="leaf in data.leafs"
        :key="data.node.name"
        :data="leaf"
        :uid="uid"
        :canChosen="canChosen"
      ></v-leaf>
    </ul>
  </li>
</template>

<script>
  import EventMixin from './mixin';
  import VLeaf from './v-leaf.vue';
  import VNode from './v-node.vue';

  export default {
    name: 'v-branch',
    mixins: [EventMixin],
    props: {
      data: {
        type: Object,
        required: true
      },
      uid: {
        type: [String, Number],
        required: true
      },
      canChosen: Boolean
    },
    components: {
      'v-node': VNode,
      'v-leaf': VLeaf
    }
  };
</script>
