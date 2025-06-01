<script setup>
import { ref, onMounted, onBeforeUnmount, computed, useAttrs } from "vue";

/* ---------- Props ---------- */
const props = defineProps({
  label:    { type: String, default: "🎉 噴彩帶！" },
  disabled: { type: Boolean, default: false },
});

/* ---------- Emits ---------- */
const emit = defineEmits(["fly"]);

/* ---------- 過濾掉會害死按鈕的 disabled 屬性 ---------- */
const attrs = useAttrs();
const forwarded = computed(() => {
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  const { disabled, ...rest } = attrs;   // ← 去掉 disabled
  return rest;
});

/* ---------- DOM 參考 ---------- */
const btn = ref(null);

/* ---------- Canvas + Confetti ---------- */
let canvas = null, ctx = null, pool = [], rafId = null;
const COLOR = ["#FFC700", "#FF0000", "#2E3192", "#41BBC7", "#7FBA00", "#F000FF"];

function ensureCanvas() {
  if (canvas) return;
  canvas = Object.assign(document.createElement("canvas"), {
    width:  window.innerWidth,
    height: window.innerHeight,
    style:  "position:fixed;inset:0;width:100%;height:100%;pointer-events:none;z-index:9999;",
  });
  ctx = canvas.getContext("2d");
  document.body.appendChild(canvas);
}
function removeCanvas() { if (canvas) { document.body.removeChild(canvas); canvas = ctx = null; } }

function makePieces(n, ox, oy) {
  const arr = [];
  for (let i = 0; i < n; i++) {
    const a = Math.random() * Math.PI * 2, v = 4 + Math.random() * 4;
    arr.push({
      x: ox, y: oy,                     // 起點＝按鈕中心
      vx: Math.cos(a) * v,
      vy: Math.sin(a) * v,
      g: 0.25 + Math.random() * 0.15,
      r: Math.random() * 360,
      rs: -10 + Math.random() * 20,
      s: 6 + Math.random() * 4,
      c: COLOR[(Math.random() * COLOR.length) | 0],
      a: 1,
    });
  }
  return arr;
}

function loop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  pool.forEach(p => {
    p.vy += p.g;
    p.x  += p.vx;
    p.y  += p.vy;
    p.r  += p.rs;
    if (p.y > canvas.height + 40) p.a -= 0.02;

    ctx.save();
    ctx.globalAlpha = p.a;
    ctx.translate(p.x, p.y);
    ctx.rotate((p.r * Math.PI) / 180);
    ctx.fillStyle = p.c;
    ctx.fillRect(-p.s / 2, -p.s / 2, p.s, p.s);
    ctx.restore();
  });
  pool = pool.filter(p => p.a > 0);

  if (pool.length) {
    rafId = requestAnimationFrame(loop);
  } else {
    cancelAnimationFrame(rafId);
    rafId = null;
    removeCanvas();
  }
}

/* ---------- 點擊事件 ---------- */
function fire() {
  if (props.disabled) return;                // 停用就不響應

  const { left, top, width, height } = btn.value.getBoundingClientRect();
  const ox = left + width  / 2 + window.scrollX;
  const oy = top  + height / 2 + window.scrollY;

  ensureCanvas();
  pool.push(...makePieces(160, ox, oy));
  if (!rafId) loop();

  emit("fly");
}

/* ---------- Resize / 清理 ---------- */
function resize() { if (canvas) { canvas.width = innerWidth; canvas.height = innerHeight; } }
onMounted(() => addEventListener("resize", resize));
onBeforeUnmount(() => {
  removeEventListener("resize", resize);
  if (rafId) cancelAnimationFrame(rafId);
  removeCanvas();
});
</script>

<template>
  <!-- 注意：只要 props.disabled 為 true 才真的加 disabled 屬性 -->
  <button
    ref="btn"
    type="button"
    :disabled="props.disabled"
    @click="fire"
    :class="[
      'px-4 py-2 rounded-xl font-semibold text-white',
      'bg-gradient-to-r from-indigo-500 to-pink-500',
      'shadow-md hover:shadow-lg transition duration-200 select-none',
      props.disabled && 'opacity-50 cursor-not-allowed',
    ]"
    v-bind="forwarded"
  >
    <slot>{{ props.label }}</slot>
  </button>
</template>

<style scoped>
button { cursor: pointer; }
</style>