<style>
@import "./autosuggest.css";
</style>
<template>
  <div class="autosuggest">
    <div class="autoinput">
      <ul class="selecteditems__list">
        <li v-for="(item, index) in selecteditems" :key="item.id">
          {{ item.name }}
          <span class="removeitem" @click="itemRemoved(index)"></span>
        </li>
      </ul>
      <span class="searchicon"></span>
      <input
        type="text"
        id="autosearch"
        v-model="query"
        @input="keypressed"
        @keydown="onkeydown"
        placeholder="Type here..."
        autocomplete="off"
      />
      <span
        :class="{
          'arrow downarrow': !this.filtereditems.length > 0,
          'arrow uparrow': this.filtereditems.length > 0,
        }"
        @click="toggleVisible"
      ></span>
    </div>
    <div class="options" v-if="this.filtereditems.length > 0">
      <ul class="list">
        <li
          v-for="item in filtereditems"
          @click="itemClicked(item.id)"
          :key="item.id"
        >
          {{ item.name }}
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
export default {
  props: ["allitems"],
  data() {
    return {
      query: "",
      visible: false,
      selecteditems: [],
      stopAutoFill: false,
      backspaceentered: false,
      searchtext: "",
    };
  },
  methods: {
    setInputSelection(input, startPos, endPos) {
      input.focus();
      if (typeof input.selectionStart != "undefined") {
        input.selectionStart = startPos;
        input.selectionEnd = endPos;
      } else if (document.selection && document.selection.createRange) {
        // IE branch
        input.select();
        var range = document.selection.createRange();
        range.collapse(true);
        range.moveEnd("character", endPos);
        range.moveStart("character", startPos);
        range.select();
      }
    },
    onkeydown: function (e) {
      this.stopAutoFill = false;

      if (e.keyCode == 8) {
        this.backspaceentered = true;
      } else if (e.keyCode == 13) {
        if (
          this.allitems.filter(
            (item) =>
              item["name"]
                .toLowerCase()
                .startsWith(e.target.value.toLowerCase(), 0) && !item.invisible
          ).length > 0
        ) {
          this.selecteditems.push(
            this.allitems.filter(
              (item) =>
                item["name"]
                  .toLowerCase()
                  .startsWith(e.target.value.toLowerCase(), 0) &&
                !item.invisible
            )[0]
          );
          var id = this.allitems.filter(
            (item) =>
              item["name"]
                .toLowerCase()
                .startsWith(e.target.value.toLowerCase(), 0) && !item.invisible
          )[0].id;
          this.allitems.forEach(function (item) {
            if (item.id === id) {
              item.invisible = true;
            }
          });

          this.query = "";
          this.filtereditems.length = 0;
        }
      }
    },
    // checkkeyentered: function (e) {
    //   console.log("e.keyCode", e.keyCode);

    // },
    keypressed: function (event) {
      this.searchtext = event.target.value;
      console.log("searchtext in keypressed", this.searchtext);
      if (this.backspaceentered) {
        this.backspaceentered = false;
      } else {
        if (!this.stopAutoFill && event.target.value != "") {
          if (
            this.allitems.filter(
              (item) =>
                item["name"]
                  .toLowerCase()
                  .startsWith(event.target.value.toLowerCase(), 0) &&
                !item.invisible
            ).length > 0
          ) {
            var querylength = event.target.value.length;
            var fullstring = this.allitems.filter(
              (item) =>
                item["name"]
                  .toLowerCase()
                  .startsWith(event.target.value.toLowerCase(), 0) &&
                !item.invisible
            )[0].name;

            event.target.value = fullstring;
            this.query = fullstring;

            this.setInputSelection(
              event.target,
              querylength,
              fullstring.length
            );
            this.stopAutoFill = true;
          }
        }
      }
    },
    toggleVisible() {
      if (this.query == "") {
        this.visible = !this.visible;
      }
    },
    itemClicked(id) {
      const found = this.selecteditems.some((item) => item.id === id);
      if (!found) {
        this.selecteditems.push(
          this.filtereditems.filter((item) => item.id === id)[0]
        );

        this.allitems.forEach(function (item) {
          if (item.id === id) {
            item.invisible = true;
          }
        });
      }
      this.query = "";
      this.filtereditems.length = 0;
    },
    itemRemoved(index) {
      console.log(this.selecteditems[index]);
      var id = this.selecteditems[index].id;
      this.allitems.forEach(function (item) {
        if (item.id === id) {
          item.invisible = false;
        }
      });
      this.selecteditems.splice(index, 1);
    },
  },
  computed: {
    filtereditems() {
      if (this.query == "") {
        if (this.visible) {
          return [...this.allitems.filter((item) => !item.invisible).slice()];
        }
        return [];
      }
      console.log("searchtext", this.searchtext);
      return this.allitems.filter(
        (item) =>
          item["name"].toLowerCase().includes(this.searchtext.toLowerCase()) &&
          !item.invisible
      );
    },
  },
  mounted() {
    this.allitems.forEach(function (item) {
      item.invisible = false;
    });
  },
};
</script>
