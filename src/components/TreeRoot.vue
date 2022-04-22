<template>
  <div>
    <div
      v-for="(pojo, idx) in json"
      :key="idx"
      draggable="true"
      @dragenter="dragenter"
      @dragend="dragend"
    >
      <div v-if="pojo.children">
        <div
          :id="pojo.id"
          class="taxonomy-item flex flex-row justify-between rounded-lg bg-white shadow-lg mb-3 font-semibold py-3 pl-10 pr-3 relative"
        >
          <span
            v-if="pojo.children && pojo.children.length > 0"
            @click="pojo.showChildren = !pojo.showChildren"
            class="absolute left-0"
          >
            <svg
              v-if="pojo.showChildren"
              xmlns="http://www.w3.org/2000/svg"
              height="24px"
              viewBox="0 0 24 24"
              width="24px"
              fill="#c4c4c4"
            >
              <path d="M0 0h24v24H0V0z" fill="none" />
              <path d="M7 10l5 5 5-5H7z" />
            </svg>
            <svg
              v-if="!pojo.showChildren"
              xmlns="http://www.w3.org/2000/svg"
              height="24px"
              viewBox="0 0 24 24"
              width="24px"
              fill="#c4c4c4"
            >
              <path d="M0 0h24v24H0V0z" fill="none" />
              <path d="M10 17l5-5-5-5v10z" />
            </svg>
          </span>
          <div>{{ pojo.name }}</div>
          <div style="color: #3896ee">{{ numberWithCommas(pojo.value) }}</div>
        </div>
      </div>
      <div class="mb-3" v-else>
        <div
          :id="pojo.key"
          draggable="true"
          @dragstart="drag"
          class="taxonomy-item flex justify-between border border-gray-400 px-3 py-2 rounded-xl mb-2 font-semibold"
        >
          <div class="flex flex-row flex-start space-x-3">
            <span>{{ pojo.key }}</span>
            <span>{{ pojo.name }}</span>
            <span style="color: #3896ee">{{
              numberWithCommas(pojo.value)
            }}</span>
          </div>
          <div class="flex flex-end">
            <span
              class="text-red-600 hover:underline cursor-pointer"
              @click="$emit('sendAddBal', pojo)"
              >{{ "clear" }}</span
            >
            <svg
              class="cursor-move"
              xmlns="http://www.w3.org/2000/svg"
              height="24px"
              viewBox="0 0 24 24"
              width="24px"
              fill="#c4c4c4"
            >
              <path d="M0 0h24v24H0V0z" fill="none" />
              <path
                d="M11 18c0 1.1-.9 2-2 2s-2-.9-2-2 .9-2 2-2 2 .9 2 2zm-2-8c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0-6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm6 4c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zm0 2c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0 6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2z"
              />
            </svg>
          </div>
        </div>
      </div>
      <div
        class="pl-3"
        v-if="pojo.children && pojo.children.length > 0 && pojo.showChildren"
      >
        <TreeRoot
          @sendAddBal="receiveAddBal"
          @sendAddBalRoot="$emit('sendAddBalRoot', $event)"
          :json="pojo.children"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TreeRoot",
  props: {
    json: {
      type: Array,
      default: () => {
        return [
          {
            id: 1,
            name: "Aktiva",
            value: 676238,
            showChildren: false,
            children: [
              {
                id: 11,
                name: "Position 1",
                value: 32.0,
                showChildren: false,
                children: [
                  {
                    id: 111,
                    name: "SubPosition 1",
                    value: 37882.0,
                    showChildren: false,
                    children: [],
                  },
                ],
              },
            ],
          },
          {
            id: 2,
            name: "Passiva",
            value: 878.36,
            showChildren: false,
            children: [],
          },
          {
            id: 3,
            name: "Gewin-und Verlustrechnung",
            value: 0.0,
            showChildren: false,
            children: [],
          },
        ];
      },
    },
    to_add: Object,
    dragended: Boolean,
  },
  data() {
    return {
      current_drag_id: "",
      current_drag_enter_id: null,
    };
  },
  computed: {
    taxonomies() {
      return [
        {
          id: 1,
          name: "Aktiva",
          value: 676238,
          showChildren: false,
          children: [
            {
              id: 11,
              name: "Position 1",
              value: 32.0,
              showChildren: false,
              children: [
                {
                  id: 111,
                  name: "SubPosition 1",
                  value: 37882.0,
                  showChildren: false,
                  children: [],
                },
              ],
            },
          ],
        },
        {
          id: 2,
          name: "Passiva",
          value: 878.36,
          showChildren: false,
          children: [],
        },
      ];
    },
  },
  methods: {
    receiveAddBal(bal) {
      const parent = this.determineParent(this.json, bal.id);
      console.log("receiveAddBal parent ]]==>> ", parent);
      if (parent) {
        const idxParent = this.json.findIndex(
          (child) => child.id === parent.id
        );
        if (idxParent >= 0) {
          const idx = this.json[idxParent].children.findIndex(
            (child) => child.id === bal.id
          );
          // eslint-disable-next-line vue/no-mutating-props
          this.json[idxParent].children.splice(idx, 1);
          this.$emit("sendAddBalRoot", bal);
        }
      } else {
        this.$emit("sendAddBal", bal);
      }
    },
    addTreeChild() {
      let tax = this.findById(parseInt(this.current_drag_enter_id), this.json);

      console.log("found element ==>>> ", tax);
      if (this.to_add && this.to_add.key) {
        tax.children.push(this.to_add);
        tax.showChildren = true;
        this.$emit("removeMe", this.to_add.key);
      }
    },
    drag(ev) {
      // ev.dataTransfer.setData('text', ev.target.id)
      this.current_drag_id = ev.target.id;
      console.log("drag id ==>> ", this.current_drag_id);
    },
    dragenter(e) {
      if (e.target.classList.contains("taxonomy-item")) {
        this.current_drag_enter_id = e.target.id;
      }
    },
    dragFinish(toAdd) {
      if (this.current_drag_enter_id >= 0) {
        let tax = this.findById(
          parseInt(this.current_drag_enter_id),
          this.json
        );

        if (tax && tax.children) {
          tax.children.push(toAdd);
          tax.showChildren = true;
          this.$emit("removeMe", toAdd.key);
          this.current_drag_enter_id = 0;
        }
      }
    },
    dragend(e) {
      if (e.target.classList.contains("taxonomy-item")) {
        console.log("the dragend ++>> ", e.target.id);
        console.log("the dragenter tree ++>> ", e.target.id);
        // console.log('current_drag_id ++>> ', this.current_drag_id)
        // console.log('current_drag_id ++>> ', typeof this.current_drag_id)
        // console.log('current_drag_enter_id ++>> ', this.current_drag_enter_id)
      }
    },
    findById(id, arr) {
      return arr.reduce((a, item) => {
        if (a) return a;
        if (item.id === id) return item;
        if (item.children) return this.findById(id, item.children);
      }, null);
    },
    determineParent(tree, cid) {
      if (tree.length === 0) {
        return false;
      }
      let parent = null;
      tree.forEach((pojo) => {
        if (pojo.children) {
          const idx = pojo.children.findIndex((child) => {
            return child.id === cid;
          });
          if (idx >= 0) {
            parent = pojo;
            return pojo;
          }
        }
      });
      return parent;
    },

    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
    numberWithCommas(x) {
      const c = x.toString().includes(".");
      x = x.toString().replace(".", "c");
      x = x.toString().replace(/\B(?<!\.\d*)(?=(\d{3})+(?!\d))/g, ",");
      return x.toString().replace("c", ".") + (!c ? ".00" : "") + " €";
    },
  },
  watch: {
    // dragended(n, o) {
    // this.sleep(20)
    // console.log('dragended watcher ==>>> ', n, o, this.current_drag_enter_id)
    // if (n && this.current_drag_enter_id) {
    //   console.log('this.current_drag_enter_id ==>> ', this.current_drag_enter_id)
    //   this.addTreeChild()
    //   this.current_drag_enter_id = null
    //   this.$emit('resetDrag')
    // }
    // },
  },
};
</script>

<style scoped></style>
