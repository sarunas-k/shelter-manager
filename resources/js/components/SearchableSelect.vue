<template>
  <div class="dropdown" v-if="options">

    <!-- Dropdown Input -->
    <input type="hidden" :name="name" :value="selected.id"/>
    <input class="dropdown-input form-control"
      @focus="showOptions()"
      @blur="exit()"
      @keyup="keyMonitor"
      v-model="searchFilter"
      :disabled="disabled"
      :placeholder="placeholder"
      autocomplete="off"
      ref="input" />

    <!-- Dropdown Menu -->
    <div class="dropdown-content"
      v-show="optionsShown">
      <div
        class="form-control dropdown-item"
        @mousedown.prevent="selectOption(option)"
        v-for="(option, index) in filteredOptions"
        :key="index">
          {{ option.name || option.id || '-' }}
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'SearchableSelect',
    template: 'SearchableSelect',
    props: {
      name: {
        type: String,
        required: false,
        default: 'dropdown',
        note: 'Input name'
      },
      options: {
        type: Array,
        required: true,
        default: [],
        note: 'Options of dropdown. An array of options with id and name',
      },
      placeholder: {
        type: String,
        required: false,
        default: 'Please select an option',
        note: 'Placeholder of dropdown'
      },
      disabled: {
        type: Boolean,
        required: false,
        default: false,
        note: 'Disable the dropdown'
      },
      maxItem: {
        type: Number,
        required: false,
        default: 100,
        note: 'Max items showing'
      },
      default: {
        required: false,
        default: null,
        note: 'Default value of input'
      }
    },
    data() {
      return {
        selected: {},
        optionsShown: false,
        searchFilter: ''
      }
    },
    mounted() {
        console.log('Vue: SearchableSelect Component mounted.');
    },
    created() {
      this.$emit('selected', this.selected);
      if (this.default)
        this.selectOption(this.options.find(option => option.id == this.default));
    },
    computed: {
      filteredOptions() {
        const filtered = [];
        const regOption = new RegExp(this.searchFilter, 'ig');
        for (const option of this.options) {
          if (this.searchFilter.length < 1 || option.name.match(regOption)){
            if (filtered.length < this.maxItem) filtered.push(option);
          }
        }
        return filtered;
      }
    },
    methods: {
      selectOption(option) {
        if (!option)
          return;
        this.selected = option;
        this.optionsShown = false;
        this.searchFilter = this.selected.name;
        this.$emit('selected', this.selected);
        if (this.$refs.input) this.$refs.input.blur();
      },
      showOptions(){
        if (!this.disabled) {
          // this.searchFilter = '';
          this.optionsShown = true;
        }
      },
      exit() {
        if (!this.selected.id) {
          this.selected = {};
          this.searchFilter = '';
        } else {
          this.searchFilter = this.selected.name;
        }
        this.$emit('selected', this.selected);
        this.optionsShown = false;
      },
      // Selecting when pressing Enter
      keyMonitor: function(event) {
        if (event.key === "Enter" && this.filteredOptions[0])
          this.selectOption(this.filteredOptions[0]);
      }
    },
    watch: {
      searchFilter() {
        if (this.filteredOptions.length === 0) {
          this.selected = {};
        } else {
          this.selected = this.filteredOptions[0];
        }
        this.$emit('filter', this.searchFilter);
      }
    }
  };
</script>


<style lang="scss" scoped>
    .dropdown-input {
      cursor: default;
    }
    .dropdown-content {
      position: absolute;
      background-color: #fff;
      width: 100%;
      border-bottom: 1px solid #e7ecf5;
      border-left: 1px solid #e7ecf5;
      border-right: 1px solid #e7ecf5;
      box-shadow: 0px -8px 34px 0px rgba(0,0,0,0.15);
      overflow: auto;
      z-index: 1;
      cursor: default;
      margin-top: 3px;
      max-height: 250px;
      .dropdown-item {
        padding-top: 7px;
        &:hover {
          background-color: #e7ecf5;
        }
      }
    }
    .dropdown:hover .dropdowncontent {
      display: block;
    }
</style>