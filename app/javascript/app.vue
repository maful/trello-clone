<template>
  <draggable v-model="lists" group="lists" class="board drag-area pb-3" @end="listMoved" draggable=".list-moved">
    <list v-for="(list, index) in lists" v-bind:key="list.id" :list="list" class="list-moved"></list>

    <div slot="footer" class="list add-list">
      <a v-if="!editing" v-on:click="startEditing"><i class="fas fa-plus"></i> Add another List</a>
      <textarea v-if="editing" ref="message" v-model="message" class="form-control mb-2" placeholder="Enter list title..."></textarea>
      <button v-if="editing" v-on:click="createList" class="btn btn-sm btn-success">Add List</button>
      <a v-if="editing" v-on:click="closeFormList" class="ml-2"><i class="fas fa-times fa-lg"></i></a>
    </div>
  </draggable>
</template>

<script>
import draggable from 'vuedraggable'
import list from 'components/list'

export default {
  components: { draggable, list },
  data: function () {
    return {
      editing: false,
      message: ""
    }
  },
  computed: {
    lists: {
      get() {
        return this.$store.state.lists
      },
      set(value) {
        this.$store.state.lists = value
      }
    }
  },
  methods: {
    startEditing: function () {
      this.editing = true
      this.$nextTick(() => { this.$refs.message.focus() })
    },
    closeFormList: function() {
      this.editing = false
      this.message = ""
    },
    listMoved: function (event) {
      let data = new FormData
      data.append("list[position]", event.newIndex + 1)

      Rails.ajax({
        url: `/lists/${this.lists[event.newIndex].id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json"
      })
    },
    createList: function () {
      let data = new FormData
      data.append("list[name]", this.message)

      Rails.ajax({
        url: "/lists",
        type: "POST",
        data: data,
        dataType: "json",
        success: data => {
          this.message = ""
          this.editing = false
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.drag-area {
  min-height: 10px;
}

.board {
  white-space: nowrap;
  overflow-x: auto;
}

.list {
  background-color: #E2E4E6;
  border-radius: 3px;
  display: inline-block;
  margin-right: 20px;
  padding: 10px;
  vertical-align: top;
  width: 270px;
}

.add-list {
  a {
    cursor: pointer;
  }

  &:hover {
    background-color: #CACCCE;
  }
}
</style>
