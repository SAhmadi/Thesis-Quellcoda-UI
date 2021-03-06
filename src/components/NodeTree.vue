<template>
  <li class="node-tree"
      :class="[node.children ? 'node-tree-directory' : '', isActive ? 'is-active' : '']"
  >
    <div v-if="node.name" :class="[node.children ? 'directory' : 'file']" class="row">
      <svg v-if="node.children"
           class="node-tree-node-icon bi bi-chevron-right"
           :class="{'chevron-open': isChevronIconOpen}"
           width="1em"
           height="1em"
           viewBox="0 0 16 16"
           fill="currentColor"
           xmlns="http://www.w3.org/2000/svg"
           @click="showNested"
      >
        <path fill-rule="evenodd" d="M4.646 1.646a.5.5 0 01.708 0l6 6a.5.5 0 010 .708l-6 6a.5.5 0 01-.708-.708L10.293 8 4.646 2.354a.5.5 0 010-.708z" clip-rule="evenodd"/>
      </svg>

      <svg v-if="!node.children" class="node-tree-node-icon file-icon bi bi-file-earmark-code" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
        <path d="M4 1h5v1H4a1 1 0 00-1 1v10a1 1 0 001 1h8a1 1 0 001-1V6h1v7a2 2 0 01-2 2H4a2 2 0 01-2-2V3a2 2 0 012-2z"/>
        <path d="M9 4.5V1l5 5h-3.5A1.5 1.5 0 019 4.5z"/>
        <path fill-rule="evenodd" d="M8.646 6.646a.5.5 0 01.708 0l2 2a.5.5 0 010 .708l-2 2a.5.5 0 01-.708-.708L10.293 9 8.646 7.354a.5.5 0 010-.708zm-1.292 0a.5.5 0 00-.708 0l-2 2a.5.5 0 000 .708l2 2a.5.5 0 00.708-.708L5.707 9l1.647-1.646a.5.5 0 000-.708z" clip-rule="evenodd"/>
      </svg>

      <!--@click="!node.children ? updateActiveNode() : ''"-->
      <div
        v-if="!showRenameInput"
        :class="[node.children ? 'directory label' : 'label']"
        @click="updateActiveNode"
        @dblclick="showRenameInput = !showRenameInput"
      >
        {{ node.name }}
      </div>

      <div
        v-if="showRenameInput"
        class="rename-input"
        @keyup.enter="renameFile"
      >
        <input
          v-model="renameInput"
          type="text"
          class="form-control"
          id="rename-file-input"
          aria-describedby="rename-file"
          placeholder=""
        >
      </div>
    </div>

    <ul v-if="node.children && node.children.length" :class="[hideNested ? 'node-tree-hide-nested' : 'node-tree-show-nested']">
      <node v-for="(child, idx) in node.children" :node="child" :key="`${idx}-${node.id}`"></node>
    </ul>
  </li>
</template>

<script>
import { updateModels } from '@/utils.js';

export default {
  name: "node",
  props: {
    'node': Object,
  },
  data: function() {
    return {
      // the node.name part is there to exclude the 'invisible' root directory
      hideNested: this.$props.node.children && this.$props.node.name,
      isChevronIconOpen: false,
      showRenameInput: false,
      renameInput: '',
    };
  },
  mounted() {
  },
  methods: {
    showNested() {
      if (!this.hideNested) {
        this.hideNested = this.$props.node.children && this.$props.node.name;
      }
      else { this.hideNested = !this.hideNested; }

      this.isChevronIconOpen = !this.isChevronIconOpen;
    },
    updateActiveNode() {
      this.$store.dispatch('updateActiveNode', this.$props.node);
    },
    renameFile() {
      this.$props.node.name = this.renameInput;
      this.showRenameInput = !this.showRenameInput;

      // Update models in Store
      updateModels(this.$props.node, this.models, this.$store);
    },
  },
  computed: {
    isActive() {
      return this.$store.getters.activeNodeId === this.$props.node.id;
    },
    models() {
      return this.$store.getters.models;
    },
  },
};
</script>

<style lang="scss">
.node-tree-directory {
  cursor: pointer;
  user-select: none;
}

.node-tree-hide-nested {
  display: none;
}
.node-tree-show-nested {
  display: block;
}

.chevron-open {
  transform: rotate(90deg);
}

.node-tree.is-active {
  color: #00AAAA;
}
.node-tree {
  list-style: none;
  margin: 10px 0 10px 6px;
  width: 100%;

  ul {
    width: 100%;
    padding: 0 16px;
  }

  .directory.label {
    font-weight: bold;
  }

  .label {
    overflow-y: hidden;
  }

  .label:hover:not(.directory) {
    color: #0C9A9A;
  }

  &-node-icon {
    font-size: 20px;
    margin: 2px 2px 0 0;
    padding: 3px;
  }
  &-node-icon:hover:not(.file-icon) {
    background: #ddd;
    border-radius: 2px;
  }

  &-action-icon {
    font-size: 20px;
    margin: 3px 0 0 6px;
    padding: 3px;
    opacity: 0.5;
  }
  &-action-icon:hover {
    background: #ddd;
    border-radius: 2px;
    opacity: 1;
  }
}
</style>