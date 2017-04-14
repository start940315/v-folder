<template>
  <ul ref="container" class="v-branch-body">
    <v-node :can-chosen="canChosen" :data="node" :uid="uid"></v-node>
    <v-branch v-show="node.open" v-for="branch in branches" :can-chosen="canChosen" :data="branch" :uid="uid"></v-branch>
    <v-leaf v-show="node.open" v-for="leaf in leafs" :can-chosen="canChosen" :data="leaf" :uid="uid"></v-leaf>
  </ul>
</template>
<script>
  import Store from './store';
  import EventMixin from './mixin';
  import VNode from './v-node.vue';
  import VLeaf from './v-leaf.vue';
  import VBranch from './v-branch.vue';
  import styles from './styles.css';

  let uid = 0;
  
  function adjustWidth (el, self) {
    var gcs = getComputedStyle;
    var pi = parseInt;
    var getHeight = (el) => pi(gcs(el).height);
    var minWidth = pi(gcs(el).width);
    var maxWidth = 500;
    var standardHeight = getHeight(el);
    var center = (minWidth+maxWidth)/2;
    el.style.width = center+"px";
    function setWidth () {
      var temp = getHeight(el);
      if( temp < standardHeight ) {
        standardHeight = temp;
        minWidth = center;
      } else {
        maxWidth = center;
      }
      if(maxWidth-minWidth > 1) {
        center = (minWidth+maxWidth)/2;
        self.$nextTick(setWidth);
        console.log(center);
      }
    }
    this.$nextTick(setWidth);
  }

  export default {
    name: 'v-folder',
    mixins: [EventMixin],
    props: {
      data: Object,
      ajax: Object,
      conf: Object,
      nowChosen: {
        type: [String, Number],
        required: true
      }
    },
    components: {
      'v-node': VNode,
      'v-leaf': VLeaf,
      'v-branch': VBranch
    },
    watch: {
      data(newVal, oldVal) {
        let nameKey = this.conf && this.conf.node || 'name';
        if (newVal[nameKey] !== oldVal[nameKey]) {
          this.store = new Store(newVal, this.conf);
        }
      }
    },
    data() {
      console.log(uid);
      return {
        uid: uid++,
        store: new Store(this.data, this.conf)
      };
    },

    computed: {
      root() {
        return this.store.dataStore;
      },
      branches() {
        return this.root.branches;
      },
      leafs() {
        return this.root.leafs;
      },
      node() {
        return this.root.node;
      },
      canChosen() {
        return this.nowChosen === this.uid;
      }
    },
    
    methods: {
      resTransform(data, node) {
        let conf = this.conf || {};
        let dirKey  = conf['branch'] || 'dirs';
        let fileKey = conf['leaf'] || 'files';
        let nameKey = conf['node'] || 'name';

        data[nameKey] = node.name;
        data[dirKey]  = data[dirKey].map(d => ({[nameKey]: d}));
        return data;
      },

      getReqConf(node) {
        let reqConf = this.ajax || {};
        let { url, method, data, params, pathAs, headers } = reqConf;

        if (method || method.toUpperCase() === 'GET') {
          reqConf.params = (params || {});
          reqConf.params[pathAs] = node.path;
        } else {
          reqConf.data = (data || {});
          reqConf.data[pathAs] = node.path;
        }

        reqConf.method = method || 'GET';
        reqConf.headers = headers || {};

        return reqConf;
      },

      request(node) {
        if (!this.ajax) {
          return Promise.reject('ajax:false');
        }

        let process = this.ajax.process || (res => res);

        return this.$http(this.getReqConf(node))
          .then(res => {
            let data = process(res.data);
            return this.resTransform(data, node);
          });
      }
    },

    created() {
      this.listen('change', node => {
        this.store.commit('change', node).then(res => this.$emit('change', res));
      });

      this.listen('unfold', node => {
        if (node.open && node.canOpen) {
          node.open =! node.open;
          adjustWidth(this.$refs["container"], this);
          return;
        }

        this.store.commit('unfold', node)
          .then(() => {

            this.request(node)
            .then(data => {
              if (data) {
                this.store.merge(data, node);
              } else {
                throw 'empty';
              }
            })
            .catch(e => {
              node.status = 'empty';
              window.console && console.error(e);
            });

          })
          .catch(e => node.status = 'done');

      });
      this.listen('choose', node => {
        this.store.commit('choose', {
          node,
          nowChosen: this.nowChosen
        }).then(res => this.$emit('choose', res));
      });
    },
    
    mounted() {
      adjustWidth(this.$refs["container"], this);
    },
    
    destroyed () {
      this.distroy();
    }
  };
</script>
