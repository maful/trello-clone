<template>
  <div class="list">
    <div class="mb-3 list-title">
      <h6>{{ list.name }}</h6>
    </div>

    <draggable v-model="list.cards" group="cards" class="drag-area" @change="cardMoved">
      <card v-for="card in list.cards" v-bind:key="card.id" :card="card" :list="list"></card>
    </draggable>

    <div class="form-card">
      <a v-if="!editing" v-on:click="startEditing"><i class="fas fa-plus"></i> Add a card</a>
      <textarea v-if="editing" ref="message" v-model="message" class="form-control mb-2" placeholder="Enter a litter card..."></textarea>
      <button v-if="editing" v-on:click="createCard" class="btn btn-sm btn-success">Add Card</button>
      <a v-if="editing" v-on:click="closeFormCard" class="ml-2"><i class="fas fa-times fa-lg"></i></a>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import card from './card'

export default {
  props: ["list"],
  components: { draggable, card },
  data: function () {
    return {
      editing: false,
      message: ""
    }
  },
  methods: {
    startEditing: function () {
      this.editing = true
      this.$nextTick(() => { this.$refs.message.focus() })
    },
    closeFormCard: function() {
      this.editing = false
      this.message = ""
    },
    cardMoved: function (event) {
      const evt = event.added || event.moved
      if (evt == undefined) { return }

      const element = evt.element
      const list_index = window.store.state.lists.findIndex((list) => {
        return list.cards.find((card) => {
          return card.id === element.id
        })
      })

      let data = new FormData
      data.append("card[list_id]", window.store.state.lists[list_index].id)
      data.append("card[position]", evt.newIndex + 1)

      Rails.ajax({
        url: `/cards/${element.id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json"
      })
    },
    createCard: function () {
      let data = new FormData
      data.append("card[list_id]", this.list.id)
      data.append("card[name]", this.message)

      Rails.ajax({
        url: "/cards",
        type: "POST",
        data: data,
        dataType: "json",
        success: data => {
          this.message = ""
          this.$nextTick(() => { this.$refs.message.focus() })
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

.list-title {
  cursor: pointer;
}

.form-card {
  border-radius: 3px;
  padding: 4px 8px;
  color: #5e6c84;

  i {
    color: #6b778c;
    &:hover {
      color: #172b4d;
    }
  }

  &:hover {
    background-color: rgba(9,30,66,.08);
    color: #172b4d;
  }

  a {
    cursor: pointer;
  }
}
</style>
