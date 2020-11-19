<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <h1>{{ count }}</h1>
  <h1>{{ double }}</h1>
  <h1 v-if="loading">loading...</h1>
  <img v-if="loaded" :src="result[0].url" />
  <ul>
    <li v-for="number in numbers" :key="number">
      <h1>{{ number }}</h1>
    </li>
  </ul>
  <h1>{{ person.name }}</h1>
  <h1>{{ greetings }}</h1>
  <button @click="increase">+1</button>
  <button @click="updateGreeting">updateGreeting</button>
  <h1>{{ x }}-{{ y }}</h1>
</template>

<script lang="ts">
// import { ref, computed} from "vue";
import {
  ref,
  reactive,
  computed,
  toRefs,
  // onMounted,
  onUpdated,
  onRenderTriggered,
  watch,
  // onUnmounted,
} from "vue";
import useMousePosition from "./hooks/useMousePosition";
import useURLLoader from "./hooks/useURLLoader";
interface DataProps {
  //定义一个类型防止报错
  count: number;
  double: number;
  increase: () => void;
  numbers: number[];
  person: { name?: string };
  // greetings: string;
  // updateGreeting: () => void;
}
interface DogResult {
  message: string;
  status: string;
}
interface CatResult {
  id: string;
  url: string;
  width: number;
  height: number;
}
export default {
  name: "App",
  setup() {
    // const count = ref(0); //ref返回的是一个响应式对象
    // const double = computed(() => {
    //   return count.value * 2;
    // });
    // const increase = () => {
    //   count.value++; //不能直接count++
    // };

    // const x = ref(0);
    // const y = ref(0);
    // const updateMouse = (e: MouseEvent) => {
    //   x.value = e.pageX;
    //   y.value = e.pageY;
    // };
    // onMounted(() => {
    //   // console.log("mounted");
    //   document.addEventListener("click", updateMouse);
    // });
    // onUnmounted(() => {
    //   document.removeEventListener("click", updateMouse);
    // });
    const { x, y } = useMousePosition(); //重用
    const { result, loading, loaded } = useURLLoader<CatResult[]>(
      "https://api.thecatapi.com/v1/images/search?limit=1"
    );
    watch(result, () => {
      if (result.value) {
        console.log("value", result.value[0].url);
      }
    });
    onUpdated(() => {
      console.log("updated");
    });
    onRenderTriggered((event) => {
      console.log(event);
    });
    const data: DataProps = reactive({
      count: 0,
      increase: () => {
        data.count++; //这里不需要使用count.value
      },
      double: computed(() => data.count * 2),
      numbers: [0, 1, 2],
      person: {},
    });
    data.numbers[0] = 5; //在vue2中不生效
    data.person.name = "emiliya"; //在vue2中不生效

    const greetings = ref("");
    const updateGreeting = () => {
      greetings.value = "hello " + greetings.value;
    };

    // document.title=greetings.value  //由于setup只执行一次，所以不会触发，此时就要用到watch
    watch([greetings, () => data.count], (newValue, oldValue) => {
      //第一个参数的类型是有要求的，不能直接写data.count   回调函数的参数分别是新值和旧值
      document.title = "updatad" + greetings.value + data.count;
      console.log(newValue);
      console.log(oldValue);
    });

    const refData = toRefs(data);
    return {
      // increase,  ··
      // double,
      // count,
      // ...data,  //注意 解构出来的类型都是普通的js类型，而不是ref响应式类型
      greetings,
      updateGreeting,
      x,
      y,
      ...refData, //这样才是响应式类型
      result,
      loading,
      loaded,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
