<template>
  <div class="vue-query-builder" :class="{ 'vue-query-builder-styled': styled }">
    <query-builder-group
      :index="0"
      :query.sync="query"
      :ruleTypes="ruleTypes"
      :rules="mergedRules"
      :maxDepth="maxDepth"
      :depth="depth"
      :styled="styled"
      :labels="mergedLabels"
      type="query-builder-group"
      ></query-builder-group>
  </div>
</template>

<script>
import QueryBuilderGroup from './components/QueryBuilderGroup.vue';
import deepClone from './utilities.js';

var defaultLabels = {
  matchType: "Тип соответствия",
  matchTypeAll: { label: "Все", value: 'All' },
  matchTypeAny: { label: "Любой", value: 'Any' },
  addRule: "Добавить",
  removeRule: "&times;",
  addGroup: "Добавить группу",
  removeGroup: "&times;",
  textInputPlaceholder: "значение",
};

export default {
  name: 'vue-query-builder',

  components: {
    QueryBuilderGroup
  },

  props: {
    rules: Array,
    labels: {
      type: Object,
      default () {
        return defaultLabels;
      }
    },
    styled: {
      type: Boolean,
      default: true
    },
    maxDepth: {
      type: Number,
      default: 3,
      validator: function (value) {
        return value >= 1
      }
    },
    value: Object
  },

  data () {
    return {
      depth: 1,
      query: {
        logicalOperator: "All",
        children: []
      },
      ruleTypes: {
        "text": {
          operators: [
            { label: 'Содержит', value: 'contains' },
            { label: 'Не содержит', value: 'does_not_contains' },
            { label: 'Равно', value: 'matches'},
            { label: 'Не равно', value: 'does_not_match' },
            { label: 'Начинается с', value: 'begins_with' },
            { label: 'Заканчивается на', value: 'ends_with' },
            { label: 'Пустой', value: 'is_empty' },
            { label: 'Не пустой', value: 'is_not_empty' },
          ],
          inputType: "text",
          id: "text-field"
        },
        "numeric": {
          operators: [
            { label: 'Равно', value: 'eq' },
            { label: 'Не равно', value: 'not_eq' },
            { label: 'Меньше', value: 'lt' },
            { label: 'Меньше или равно', value: 'lteq' },
            { label: 'Больше', value: 'gt' },
            { label: 'Больше или равно', value: 'gteq' },
            { label: 'Пустой', value: 'is_empty' },
            { label: 'Не пустой', value: 'is_not_empty' },
          ],
          inputType: "number",
          id: "number-field"
        },
        "date": {
          operators: [
            { label: 'Равно', value: 'eq' },
            { label: 'Не равно', value: 'not_eq' },
            { label: 'Меньше', value: 'lt' },
            { label: 'Меньше или равно', value: 'lteq' },
            { label: 'Больше', value: 'gt' },
            { label: 'Больше или равно', value: 'gteq' },
            { label: 'Пустой', value: 'is_empty' },
            { label: 'Не пустой', value: 'is_not_empty' },
          ],
          inputType: "date",
          id: "date-field"
        },
        "datetime": {
          operators: [
            { label: 'Равно', value: 'eq' },
            { label: 'Не равно', value: 'not_eq' },
            { label: 'Меньше', value: 'lt' },
            { label: 'Меньше или равно', value: 'lteq' },
            { label: 'Больше', value: 'gt' },
            { label: 'Больше или равно', value: 'gteq' },
            { label: 'Пустой', value: 'is_empty' },
            { label: 'Не пустой', value: 'is_not_empty' },
          ],
          inputType: "datetime-local",
          id: "datetime-field"
        },
        "custom": {
          operators: [],
          inputType: "text",
          id: "custom-field"
        },
        "radio": {
          operators: [],
          choices: [],
          inputType: "radio",
          id: "radio-field"
        },
        "checkbox": {
          operators: [],
          choices: [],
          inputType: "checkbox",
          id: "checkbox-field"
        },
        "select": {
          operators: [
            { label: 'Равно', value: 'eq' },
            { label: 'Не равно', value: 'not_eq' }
          ],
          choices: [],
          inputType: "select",
          id: "select-field"
        },
        "multi-select": {
          operators: [{ label: 'Равно', value: 'eq' }],
          choices: [],
          inputType: "select",
          id: "multi-select-field"
        },
      }
    }
  },

  computed: {
    mergedLabels () {
      return Object.assign({}, defaultLabels, this.labels);
    },

    mergedRules () {
      var mergedRules = [];
      var vm = this;

      vm.rules.forEach(function(rule){
        if ( typeof vm.ruleTypes[rule.type] !== "undefined" ) {
          mergedRules.push( Object.assign({}, vm.ruleTypes[rule.type], rule) );
        } else {
          mergedRules.push( rule );
        }
      });

      return mergedRules;
    }
  },

  mounted () {
    this.$watch(
      'query',
      newQuery => {
        this.$emit('input', deepClone(newQuery));
      }, {
      deep: true
    });

    if ( typeof this.$options.propsData.value !== "undefined" ) {
      this.query = Object.assign(this.query, this.$options.propsData.value);
    }
  }
}
</script>

<style>
  .vue-query-builder-styled .vqb-group .rule-actions {
    margin-bottom: 20px;
  }

  .vue-query-builder-styled .vqb-rule {
    margin-top: 15px;
    margin-bottom: 15px;
    background-color: #f5f5f5;
    border-color: #ddd;
    padding: 15px;
  }

  .vue-query-builder-styled .vqb-rule label {
    margin-right: 10px;
  }

  .vue-query-builder-styled .vqb-group.depth-1 .vqb-rule,
  .vue-query-builder-styled .vqb-group.depth-2 {
    border-left: 2px solid #8bc34a;
  }

  .vue-query-builder-styled .vqb-group.depth-2 .vqb-rule,
  .vue-query-builder-styled .vqb-group.depth-3 {
    border-left: 2px solid #00bcd4;
  }

  .vue-query-builder-styled .vqb-group.depth-3 .vqb-rule,
  .vue-query-builder-styled .vqb-group.depth-4 {
    border-left: 2px solid #ff5722;
  }

  .vue-query-builder-styled .close {
    opacity: 1;
    color: rgb(150,150,150);
  }

  @media (min-width: 768px) {
    .vue-query-builder-styled .vqb-rule.form-inline .form-group {
      display: block;
    }
  }
</style>
