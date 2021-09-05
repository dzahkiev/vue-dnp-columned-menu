<template>
  <div class="card-scene">
    <Container
        orientation="horizontal"
        @drop="onColumnDrop($event)"
        drag-handle-selector=".column-drag-handle"
        @drag-start="dragStart"
        :drop-placeholder="upperDropPlaceholderOptions"
    >
      <Draggable v-for="column in scene.children" :key="column.id">
        <div class="card-container">
          <div class="card-column-header">
            <span class="column-drag-handle">&#x2630;</span>
            {{ column.name }}
            <b-button
                size="sm"
                class="add-new-btn"
                variant="success"
                v-b-modal.modal-prevent-closing
                @click="setCurrent(column.id)"
            >
              <b-icon-plus></b-icon-plus>
            </b-button>
          </div>
          <Container
              group-name="col"
              @drop="(e) => onCardDrop(column.id, e)"
              @drag-start="(e) => log('drag start', e)"
              @drag-end="(e) => log('drag end', e)"
              :get-child-payload="getCardPayload(column.id)"
              drag-class="card-ghost"
              drop-class="card-ghost-drop"
              :drop-placeholder="dropPlaceholderOptions"
          >
            <Draggable v-for="card in column.children" :key="card.id">
              <div class="card" style="background: #fff">
                <p style="float: left">
                  <span :class="{'label-type': card.type === 'label'}">{{ card.ru.title }}</span>
                  <span>
                      <b-icon-trash-fill
                          style="float: right;cursor: pointer"
                          variant="danger"
                          @click="deleteItem(column.id, card.id)"
                          title="Удалить">
                      </b-icon-trash-fill>
                      <b-icon-pencil-fill
                          style="float: right;cursor: pointer"
                          variant="primary"
                          v-b-modal.modal-prevent-closing
                          @click="setCurrent(column.id, card)"
                          title="Редактировать">
                      </b-icon-pencil-fill>
                    </span>
                </p>
              </div>
            </Draggable>
          </Container>
        </div>
      </Draggable>
    </Container>

    <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Редактирование пункта меню"
        @show="resetModal"
        @hidden="resetModal"
        @ok="handleOk"
        size="lg"
    >
      <form ref="form" @submit.stop.prevent="handleSubmit">
        <b-form-group label="Вид пункта меню" v-slot="{ ariaDescribedby }">
          <b-form-radio-group
              id="radio-group-2"
              v-model="modal.selected"
              :aria-describedby="ariaDescribedby"
              name="radio-sub-component"
          >
            <b-form-radio
                name="some-radios"
                v-model="modal.selected"
                value="label">
              простой заголовок
            </b-form-radio>
            <b-form-radio
                v-model="modal.selected"
                name="some-radios"
                value="link">
              ссылка
            </b-form-radio>
          </b-form-radio-group>
        </b-form-group>
        <div>
          <b-tabs content-class="mt-3" align="right" style="margin-top: -40px">
            <b-tab v-for="language in languages" :key="language.code"  :title="language.title" :active="language.active">
              <b-form-checkbox v-model="modal[language.code].isActive" switch>
                Активность
              </b-form-checkbox>
              <br>
              <b-form-group
                  label="Заголовок"
                  invalid-feedback="Это поле обязательно для заполнения"
                  :state="modal[language.code].titleState"
              >
                <b-form-input
                    v-model="modal[language.code].title"
                    :state="modal[language.code].titleState"
                    @input="changeActivity(language.code)"
                ></b-form-input>
              </b-form-group>
              <b-form-group
                  label="Ссылка"
                  label-for="link-input${language.code}"
                  invalid-feedback="Это поле обязательно для заполнения"
                  :state="modal[language.code].linkState"
              >
                <b-form-input
                    v-model="modal[language.code].link"
                    :state="modal[language.code].linkState"
                    :disabled="modal.selected !== 'link'"
                    @input="changeActivity(language.code)"
                ></b-form-input>
              </b-form-group>
            </b-tab>
          </b-tabs>
        </div>
      </form>
    </b-modal>
    <div>
      <b-form-textarea
          id="textarea"
          :name="fieldName"
          hidden
          :value="JSON.stringify(scene.children, undefined, 2)"
          rows="3"
          max-rows="25"
      ></b-form-textarea>

      <div class="json-preview">
        <p><b>JSON preview:</b></p>
        <div>
          {{JSON.stringify(scene.children, undefined, 2)}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Container, Draggable } from 'vue-smooth-dnd'

const languages = [
  {
    code: 'ru',
    title: 'Русский',
    active: true,
  },
  {
    code: 'en',
    title: 'English',
    active: false,
  }
];

const modalDefaultState = {
  columnId: null,
  itemId: null,
  selected: 'link',
  ru: {
    isActive: true,
    title: '',
    link: '',
    titleState: null,
    linkState: null,
  },
  en: {
    isActive: false,
    title: '',
    link: '',
    titleState: null,
    linkState: null,
  }
}

export default {
  components: {Container, Draggable},
  name: 'SortableColumnedMenu',
  props: {
    fieldName: {type: String, required: true},
    columns: {type: Array, default: () => {
      let res = [];
      for (let i = 1; i <= 4; i++) {
        res.push({
          id: i,
          name: `Колонка #${i}`,
          children: []
        });
      }
      return res;
    }},
  },
  data () {
    return {
      languages,
      scene: {children: this.columns},
      upperDropPlaceholderOptions: {
        className: 'cards-drop-preview',
        animationDuration: '150',
        showOnTop: true
      },
      dropPlaceholderOptions: {
        className: 'drop-preview',
        animationDuration: '150',
        showOnTop: true
      },
      modalDefaultState,
      modal: {...modalDefaultState}
    }
  },
  methods: {
    onColumnDrop (dropResult) {
      const scene = Object.assign({}, this.scene)
      scene.children = this.applyDrag(scene.children, dropResult)
      this.scene = scene
    },
    onCardDrop (columnId, dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const scene = Object.assign({}, this.scene)
        const column = scene.children.filter(p => p.id === columnId)[0]
        const columnIndex = scene.children.indexOf(column)
        const newColumn = Object.assign({}, column)
        newColumn.children = this.applyDrag(newColumn.children, dropResult)
        scene.children.splice(columnIndex, 1, newColumn)
        this.scene = scene
      }
    },
    getCardPayload (columnId) {
      return index => {
        return this.scene.children.filter(p => p.id === columnId)[0].children[index]
      }
    },

    applyDrag (arr, dragResult) {
      const { removedIndex, addedIndex, payload } = dragResult
      if (removedIndex === null && addedIndex === null) return arr

      const result = [...arr]
      let itemToAdd = payload

      if (removedIndex !== null) {
        itemToAdd = result.splice(removedIndex, 1)[0]
      }

      if (addedIndex !== null) {
        result.splice(addedIndex, 0, itemToAdd)
      }

      return result;
    },

    dragStart () {
      console.log('drag started')
    },

    deleteItem(columnId, itemId) {
      const column = this.scene.children.filter(p => p.id === columnId)[0];
      column.children.splice(column.children.findIndex(e => e.id === itemId),1);
    },

    updateItem(columnId, itemId) {
      console.log('update')
      const column = this.scene.children.filter(p => p.id === columnId)[0];
      const item = column.children.filter(e => e.id === itemId)[0];
      item.type = this.modal.selected;
      const that = this;
      languages.forEach(function (lang) {
        item[lang.code].link = that.modal.selected === 'link' ? that.modal[lang.code].link : '';
        item[lang.code].title = that.modal[lang.code].title;
        item[lang.code].isActive = that.modal[lang.code].isActive;
      })
      console.log(item)
    },

    addItem(columnId) {
      console.log(columnId)
      const column = this.scene.children.filter(p => p.id === columnId)[0];
      let saveLnk = this.modal.selected === 'link';
      column.children.push({
        id: `${columnId}.${column.children.length + 1}`,
        type: this.modal.selected,
        ru: {
          title: this.modal.ru.title,
          link:  saveLnk ? this.modal.ru.link : '',
          isActive: this.modal.ru.isActive,
        },
        en: {
          title: this.modal.en.title,
          link:  saveLnk ? this.modal.en.link : '',
          isActive: this.modal.en.isActive,
        }
      })
    },

    checkFormValidity() {
      let valid = true;
      const that = this;
      this.languages.forEach(function (lang) {
        if (that.modal[lang.code].isActive === true) {
          valid = valid && (that.modal[lang.code].titleState = that.modal[lang.code].title !== '');
          valid = valid && (that.modal[lang.code].linkState = that.modal.selected !== 'link' || that.modal[lang.code].link !== '');
        }
      })
      return valid;
    },
    resetModal() {
      Object.assign(this.modal.ru, {
        titleState: null,
        linkState: null,
      })
      Object.assign(this.modal.en, {
        titleState: null,
        linkState: null,
      })
    },
    handleOk(bvModalEvt) {
      bvModalEvt.preventDefault()
      this.handleSubmit()
    },
    handleSubmit() {
      // Exit when the form isn't valid
      if (!this.checkFormValidity()) {
        return
      }

      if (!this.modal.itemId) {
        this.addItem(this.modal.columnId);
      } else {
        this.updateItem(this.modal.columnId, this.modal.itemId)
      }

      // Hide the modal manually
      this.$nextTick(() => {
        this.$bvModal.hide('modal-prevent-closing')
      })
    },

    setCurrent(columnId, card) {
      Object.assign(this.modal, this.modalDefaultState);
      this.modal.columnId = columnId;
      this.modal.itemId = null;
      this.modal.ru = {...this.modal.ru, ...{title: '', link: ''}}
      this.modal.en = {...this.modal.en, ...{title: '', link: ''}}

      if (card) {
        this.modal = Object.assign(this.modal, {
          itemId: card.id,
          selected: card.type,
          ru: {...card.ru},
          en: {...card.en}
        });
      }
      console.log(this.modal.ru.title)

    },
    changeActivity(lang) {
      this.modal[lang].isActive = true;
    }
  }
}
</script>
<style>
.card-scene {
  padding: 50px;
}
.card-container {
  margin: 5px;
  width: 320px;
  min-height: 100px;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.12), 0 1px 1px rgba(0, 0, 0, 0.24);
  position: relative;
  display: flex;
  flex-direction: column;
  word-wrap: break-word;
  background-color: #fff;
  background-clip: border-box;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
  border-radius: 0;
}
.card {
  margin-top: 8px;
  background-color: white;
  padding: 5px 5px 0 5px;
  border: none!important;
  border-radius: 0!important;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125) !important;
}
.card-column-header {
  padding: .75rem 1.25rem;
  margin-bottom: 0;
  background-color: rgba(0,0,0,.03);
  border-bottom: 1px solid rgba(0,0,0,.125);
}
.column-drag-handle {
  cursor: move;
  padding: 5px;
}
.card-ghost {
  transition: transform 0.18s ease;
  transform: rotateZ(5deg);
}
.modal-lg {
  max-width: 650px!important;
}
.add-new-btn {
  float:right;margin-right:-10px;
}
.label-type {
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: bold;
  color: #c0c0c0;
}
.json-preview {
  margin:30px;
  font-family:'Courier New';
}
.json-preview div {
  max-height: 600px;
  overflow-y: scroll;
  font-size: 12px;
  white-space:pre;
}
</style>