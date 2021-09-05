<template>
  <sortable-columned-menu
      field-name="test"
      :columns="columns"
  ></sortable-columned-menu>
</template>



<script>
import SortableColumnedMenu from "@/components/SortableColumnedMenu";
const columns = [
  {
    id: 1,
    name: 'Колонка #1',
    children: [
      {
        id: 1,
        type: 'label',
        ru: {
          title: 'Заголовок',
          link: '',
          isActive: true
        },
        en: {
          title: 'Label',
          link: '',
          isActive: true
        }
      },
      {
        id: 2,
        type: 'link',
        ru: {
          title: 'Ссылка меню',
          link: 'http://',
          isActive: true
        },
        en: {
          title: 'Link',
          link: 'http://',
          isActive: true
        }
      }
    ]
  },
  {
    id: 2,
    name: 'Колонка #2',
    children: [
      {
        id: 1,
        type: 'label',
        ru: {
          title: 'Заголовок',
          link: '',
          isActive: true
        },
        en: {
          title: 'Label',
          link: '',
          isActive: true
        }
      },
      {
        id: 2,
        type: 'link',
        ru: {
          title: 'Ссылка меню',
          link: 'http://',
          isActive: true
        },
        en: {
          title: 'Menu 2',
          link: 'http://',
          isActive: true
        }
      }
    ]
  },
  {
    id: 3,
    name: 'Колонка #3',
    children: []
  },
  {
    id: 4,
    name: 'Колонка #4',
    children: []
  }
]

export default {
  components: {SortableColumnedMenu},
  data() {
    return {
      columns
    };
  }
}
</script>
