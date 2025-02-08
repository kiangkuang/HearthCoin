<script setup lang="ts">
import defeat from "@/assets/defeat.png";
import victory from "@/assets/victory.png";
import {
  useDraggable,
  useElementBounding,
  useElementSize,
  useTimeoutFn,
  whenever,
} from "@vueuse/core";
import { ref, useTemplateRef } from "vue";

const board = useTemplateRef<HTMLElement>("board");
const coin = useTemplateRef<HTMLElement>("coin");
const dmg = useTemplateRef<HTMLElement>("dmg");

const { width, height, left, top } = useElementBounding(board);
const { width: coinWidth, height: coinHeight } = useElementSize(coin);

whenever(
  () => board.value && height.value && coinWidth.value && coinHeight.value,
  () => reset(),
);

const disableDrag = ref(false);
const { style, isDragging, position } = useDraggable(coin, {
  preventDefault: true,
  initialValue: { x: screen.width * 0.7, y: screen.height * 0.5 },
  onMove(position) {
    position.x += coinWidth.value / 2 - left.value;
    position.y -= top.value;
  },
  onEnd(position) {
    if (position.y > height.value * 0.5) {
      reset();
    } else {
      disableDrag.value = true;
      showCoin.value = false;
      if (Math.random() < 0.5) {
        result(true);
      } else {
        result(false);
      }
    }
  },
  disabled: disableDrag,
});

const showCoin = ref(false);
const showDmg = ref(false);
const isVictory = ref(true);
const showResult = ref(false);
const canResultReset = ref(false);

function result(victory: boolean) {
  useTimeoutFn(() => {
    showDmg.value = true;

    useTimeoutFn(() => {
      showDmg.value = false;

      useTimeoutFn(() => {
        isVictory.value = victory;
        showResult.value = true;

        position.value = {
          x: screen.width * 0.7,
          y: screen.height * 0.5,
        };

        useTimeoutFn(() => {
          canResultReset.value = true;
        }, 300);
      }, 300);
    }, 1000);
  }, 300);
}

function reset() {
  showResult.value = false;
  showDmg.value = false;
  position.value = {
    x: width.value * 0.5,
    y: height.value * 0.7,
  };
  showCoin.value = true;
  disableDrag.value = false;
  canResultReset.value = false;
}
</script>

<template>
  <div class="flex h-screen w-screen items-center justify-center bg-gray-900 select-none">
    <div
      class="relative flex h-full max-h-[calc(100vw*9/16)] w-full max-w-[calc(100vh*16/9)] items-center justify-center"
    >
      <img ref="board" src="@/assets/board.jpg" class="h-full" />
      <img
        ref="coin"
        src="@/assets/coin-g.png"
        class="absolute h-[30%] translate-x-[-50%] transition-all ease-in-out"
        :class="{
          'duration-75': isDragging,
          'duration-500': !isDragging,
          'scale-150 opacity-0': !showCoin,
        }"
        :style="style"
      />
      <img
        ref="dmg"
        src="@/assets/dmg.png"
        class="absolute h-[20%] transition-[opacity,scale] duration-300 ease-in-out"
        :class="{
          'scale-50 opacity-0': !showDmg,
          'scale-100 opacity-100': showDmg,
          'top-[10%]': isVictory,
          'top-[67%]': !isVictory,
        }"
      />
      <img
        @click="canResultReset && reset()"
        :src="isVictory ? victory : defeat"
        class="absolute top-[50%] h-[80%] translate-y-[-50%] transition-[opacity,scale] duration-300 ease-in-out"
        :class="{
          'scale-50 opacity-0': !showResult,
          'scale-100 opacity-100': showResult,
        }"
      />
    </div>
  </div>
</template>
