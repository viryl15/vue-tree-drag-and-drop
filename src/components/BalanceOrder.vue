<template>
  <div class="flex flex-row">
    <div
      id="bal-list"
      draggable="true"
      @dragover="dragover"
      @dragleave="dragleave"
      @dragend="dragend"
      @dragenter="dragenter"
      class="flex flex-col pt-4 w-1/2"
    >
      <div
        :id="bal.key"
        draggable="true"
        @dragstart="drag"
        v-for="(bal, idx) in balance_rows"
        :key="idx"
        class="bal-item flex justify-between border border-gray-400 px-3 py-2 rounded-xl mb-2 font-semibold"
      >
        <div class="flex flex-row flex-start space-x-3">
          <div>{{ bal.key }}</div>
          <div>{{ bal.name }}</div>
        </div>
        <div class="flex flex-end" style="color: #3896ee">
          <span>{{ numberWithCommas(bal.value) }}</span>
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
    <div class="ml-4 w-1/2">
      <TreeRoot
        ref="treeroot"
        :to_add="dragelt"
        :dragended="dragended"
        :json="json"
        @resetDrag="dragended = false"
        @removeMe="removeMe($event)"
        @sendAddBalRoot="addBal"
      />
    </div>
  </div>
</template>

<script>
import TreeRoot from "@/components/TreeRoot.vue";

export default {
  name: "BalanceOrder",
  components: { TreeRoot },
  data() {
    return {
      current_drag_id: "",
      current_drag_enter_id: "",
      dragelt: null,
      dragended: false,
      balance_rows: [
        {
          id: 5326,
          key: 5326,
          name: "Zeile 1",
          value: 23156.334,
        },
        {
          id: 345,
          key: 345,
          name: "Gohze",
          value: 6098.334,
        },
        {
          id: 88,
          key: 88,
          name: "Zeile 2",
          value: 35.334,
        },
        {
          id: 777,
          key: 777,
          name: "RODYL",
          value: 897.334,
        },
      ],
      json: [
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
      ]
    };
  },
  methods: {
    addBal(bal) {
      console.log("bal to add ==>> ", bal);
      this.balance_rows.push(bal);
    },
    removeMe(id) {
      let toRemove = this.balance_rows.findIndex((elt) => elt.key === id);
      if (toRemove >= 0) this.balance_rows.splice(toRemove, 1);
    },
    drag(ev) {
      // ev.dataTransfer.setData('text', ev.target.id)
      this.current_drag_id = ev.target.id;
      // console.log('drag id ==>> ', this.current_drag_id)
    },
    // dragover(e) {
    //   if (e.target.classList.contains('bal-item')) {
    //     console.log('the dragover ++>> ', e.target.id)
    //   }
    // },
    dragleave(e) {
      if (e.target.classList.contains("bal-item")) {
        // console.log('the dragleave ++>> ', e.target.id)
        // console.log('current_drag_id ++>> ', this.current_drag_id)
      }
    },
    dragenter(e) {
      if (e.target.classList.contains("bal-item")) {
        // console.log('the dragenter ++>> ', e.target.id)
        this.current_drag_enter_id = e.target.id;
        // console.log('current_drag_enter_id ++>> ', this.current_drag_enter_id)
      }
    },
    dragend(e) {
      // for balance list
      if (e.target.classList.contains("bal-item")) {
        if (this.current_drag_enter_id) {
          let toshiftIindex = this.balance_rows.findIndex(
            (elt) => "" + elt.key === this.current_drag_enter_id
          );
          // check element to shift exit
          if (toshiftIindex >= 0) {
            const oldDrageltIndex = this.balance_rows.findIndex(
              (elt) => "" + elt.key === this.current_drag_id
            );
            // remove drag element
            if (oldDrageltIndex >= 0) {
              const dragelt = this.balance_rows[oldDrageltIndex];
              this.dragelt = dragelt;
              this.balance_rows.splice(oldDrageltIndex, 1);
              // get index of element to shift
              let toshiftIindex = this.balance_rows.findIndex(
                (elt) => "" + elt.key === this.current_drag_enter_id
              );
              if (toshiftIindex >= 0) {
                if (toshiftIindex >= oldDrageltIndex) {
                  // move to top
                  this.balance_rows.splice(toshiftIindex, 0, dragelt);
                  let newToShiftIdx = this.balance_rows.findIndex(
                    (elt) => "" + elt.key === this.current_drag_enter_id
                  );
                  let newDrageltIndex = this.balance_rows.findIndex(
                    (elt) => "" + elt.key === this.current_drag_id
                  );
                  let toshiftElt = this.balance_rows[newToShiftIdx];
                  this.balance_rows[newToShiftIdx] =
                    this.balance_rows[newDrageltIndex];
                  this.balance_rows[newDrageltIndex] = toshiftElt;
                } else {
                  // move to bottom
                  this.balance_rows.splice(toshiftIindex, 0, dragelt);
                }
              } else {
                this.balance_rows.splice(oldDrageltIndex, 0, dragelt);
              }
            }
          }
        }
      }
      //  for taxonomies
      if (e.target.classList.contains("taxonomy-item")) {
        console.log("taxonomies the dragend ++>> ", e.target.id);
      }
      this.dragended = true;
      const oldDrageltIndex = this.balance_rows.findIndex(
        (elt) => "" + elt.key === this.current_drag_id
      );
      if (oldDrageltIndex >= 0) {
        this.$refs.treeroot.dragFinish(this.balance_rows[oldDrageltIndex]);
      }
    },
    numberWithCommas(x) {
      const c = x.toString().includes(".");
      x = x.toString().replace(".", "c");
      x = x.toString().replace(/\B(?<!\.\d*)(?=(\d{3})+(?!\d))/g, ",");
      return x.toString().replace("c", ".") + (!c ? ".00" : "") + " €";
    },
  },
  mounted() {
    // function DragOver (ev) {
    //   ev.preventDefault()
    // }
    // function Drop (ev) {
    //   ev.preventDefault()
    //   const data = ev.dataTransfer.getData('text')
    //   console.log('data ==>> ', data)
    //   console.log('target ==>> ', ev.target)
    //   console.log('this.current_drag_id ==>> ', this.current_drag_id)
    //   ev.target.appendChild(document.getElementById(data))
    // }
    // const elemens = document.querySelectorAll('.draggable');
    // [].forEach.call(elemens, function (elem) {
    //   elem.addEventListener('dragover', DragOver, false)
    //   elem.addEventListener('drop', Drop, false)
    // })
  },
};
</script>

<style scoped>
#divContenedor {
  width: 950px;
  height: 500px;
}

#div1,
#div2,
#div3 {
  width: 350px;
  height: 70px;
  border: 1px solid #aaaaaa;
  background-color: green;
}

#div4 {
  width: 350px;
  height: 70px;
  border: 1px solid #aaaaaa;
  background-color: blue;
}
</style>
