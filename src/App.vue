<template>
  <div id="app">
    <span>Add before <em>{{task_first}}</em></span>
    <input type="text" v-model='add_before' @keyup.enter='do_add_before(task_first)'>
    <ol>
      <li v-for='task in tasks' :key="task">
        {{task}} 
        <button @click='close_task(task)'>x</button>
        <button @click='do_add_after(task)'>+</button>
      </li>
    </ol>
    <span>Add after <em>{{task_last}}</em></span>
    <input type="text" v-model='add_after' @keyup.enter='do_add_after(task_last)'>
    <p>
      <button @click='reset'>Reset</button>
    </p>

    <h3>Start with</h3>
    {{sources.join(',')}}

    <h3>End up completing</h3>
    {{sinks.join(',')}}
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld";
import { Graph, alg as Alg } from "graphlib";

let graph;

export default {
  name: "App",
  components: {
    HelloWorld
  },
  data() {
    return {
      tasks: [],
      add_before: "",
      add_after: "",
    };
  },
  computed: {
    task_first() {
      return this.tasks[0];
    },
    task_last() {
      return this.tasks[this.tasks.length - 1];
    },
    sources() {
      return graph.sources();
    },
    sinks() {
      return graph.sinks();
    }
  },
  mounted() {
    this.reset();
    this.update_graph();
  },
  methods: {
    update_graph() {
      const first = graph.sources()[0];
      // console.log(graph.edges(), graph.nodes(), graph.sources(), graph.sinks());
      // console.log("first", first)
      const new_tasks = Alg.preorder(graph, first);
      // const new_tasks = graph.nodes();
      // console.log("new_tasks", new_tasks);
      this.tasks = new_tasks;
    },
    do_add_before(task) {
      // console.log(graph, "add", this.add_before, "before", task);
      graph.setNode(this.add_before);
      // console.log(graph.nodes());
      graph.setEdge(this.add_before, task);
      // console.log(graph.edges());
      this.add_before = "";
      this.update_graph();
    },
    do_add_after(task) {
      if (task == this.task_last) {
        console.log(graph, "add", this.add_after, "after", task);
        graph.setNode(this.add_after);
        graph.setEdge(task, this.add_after);
      } else {
        console.log('do it anbothewe way')
        this.insert_task()
      }
      this.add_after = "";
      this.update_graph();
    },
    insert_task(before_task, new_task) {
      const pred = graph.predecessors(before_task);
      const succ = graph.successors(before_task);
      if (pred.length && succ.length) {
        
      }
    },
    close_task(node) {
      const pred = graph.predecessors(node);
      const succ = graph.successors(node);
      if (pred.length && succ.length) {
        graph.setEdge(pred, succ);
      }
      graph.removeNode(node);
      this.update_graph();
    },
    reset() {
      graph = new Graph();
      const tasks = ["open box", "pour cereal", "eat"];
      tasks.forEach((task, index) => {
        graph.setNode(task);
        if (index > 0 && index < tasks.length) {
          const previous_task = tasks[index - 1];
          graph.setEdge(previous_task, task);
        }
      });
      this.update_graph();
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  margin-top: 60px;
}

</style>
