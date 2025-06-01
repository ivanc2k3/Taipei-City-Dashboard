<script setup>
import { ref, computed, defineProps, defineEmits, useAttrs } from 'vue'

/* ─── Props & Emits ─── */
const props = defineProps({
  label:    { type: String, default: '酷知識' },
  size:     { type: Number, default: 120 },      // 直徑
  disabled: { type: Boolean, default: false },
  chart_config: Object,
  activeChart: String,
  series: Object,
  map_config: Object,
  map_filter: Object,
  map_filter_on: Boolean,
})


const emit = defineEmits(['click'])

/* ─── Forward non-conflicting attrs ─── */
const attrs = useAttrs()
const forwarded = computed(() => {
  const { disabled, ...rest } = attrs
  return rest
})

const showDialog = ref(false)

function handleClick (e) {
  if (props.disabled) return
  emit('click', e)          // 保留對外事件
  showDialog.value = true    // 打開彈窗
}
function closeDialog () { showDialog.value = false }

</script>

<template>
  <!-- 這個包裝層會把按鈕拉到父層正中央 -->
  <div class="neon-wrapper">
    <button
        v-show="!showDialog"
        class="neon-circle"
        :style="`--diameter:${props.size}px`"
        :disabled="props.disabled"
        @click="handleClick"
        v-bind="forwarded"
    >
    <span class="label" :data-text="props.label">{{ props.label }}</span>
    </button>
    <div
      v-if="showDialog"
      class="dialog-backdrop"
      @click.self="closeDialog"
    >
    <center><div class="dialog">
        <h2 class="dialog-title">酷知識</h2>
        </div>
        </center>
            <div
    v-if="activeChart === 'TextUnitChart'"
    class="TextUnitChart"
  >
    <div class="TextUnitChart__container">
      <div
        v-for="item in series"
        :key="item.name"
        class="TextUnitChart__content"
      >
        <div
          class="TextUnitChart__name"
          :style="{ color: props.chart_config.color[0] }"
        >
          {{ item.name }}
        </div>
        <div>
          <span
            class="TextUnitChart__value"
            :style="{ color: props.chart_config.color[1] }"
          >{{ item.data[0] }}</span>
          <span
            class="TextUnitChart__unit"
            :style="{ color: props.chart_config.color[2] }"
          >{{ item.icon }}</span>
        </div>
      </div>
    </div>
  </div>
        <br/>
        <div class="dialog">
        <center><button class="dialog-btn" @click="closeDialog">酷！</button></center>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">

.TextUnitChart {
	position: relative;
	max-height: 100%;
	height: 100%;
	flex: 1;
	color: var(--color-normal-text);
	overflow-y: auto;

	&__container {
		display: grid;
		grid-template-columns: 1fr 1fr;
		min-height: 100%
	}
	&__content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 1rem;
		border-bottom: 1px solid var(--color-border);

		// 右邊框（不包括每行最後一個）
		&:not(:nth-child(2n)) {
			border-right: 1px solid var(--color-border);
		}
    
		// 移除最後一個項目的底部邊框
		&:last-child {
			border-bottom: none;
		}
    
		// 倒數第二個如果在右邊（偶數位置），移除底部邊框
		&:nth-last-child(2):nth-child(2n-1) {
			border-bottom: none;
		}
	}
	&__value {
		font-size: 1.5rem;
		padding-right: 0.25rem;
	}
}

/* ────────────────────────────────────────────── */
/*  對齊父層中央的包裝層                             */
/* ────────────────────────────────────────────── */
.neon-wrapper {
  display: flex;
  align-items: center;   /* 垂直置中 */
  justify-content: center;/* 水平置中 */
  width: 100%;
  height: 100%;
}

/* ────────────────────────────────────────────── */
/*  原本的「霓虹圓形按鈕」                          */
/* ────────────────────────────────────────────── */
.neon-circle {
  width:            var(--diameter);
  height:           var(--diameter);
  border-radius:    50%;
  border:           4px solid #259cc3;
  background:       #222327;
  color:            #259cc3;
  font-family:      'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size:        calc(var(--diameter) * 0.1);
  letter-spacing:   2px;
  text-transform:   uppercase;

  display: flex;
  align-items: center;
  justify-content: center;

  position: relative;
  cursor: pointer;
  user-select: none;
  transition: transform 0.15s ease-in-out;

  /* 霓虹外／內光暈 */
  box-shadow:
    0 0 10px  #259cc3,
    0 0 20px  #259cc3,
    0 0 40px  #259cc3,
    /* 0 0 40px  #259cc3, */
    0 0 60px  #222327,
    /* 0 0 80px  #259cc3, */
    inset 0 0 10px #259cc3,
    inset 0 0 20px #259cc3;
}

/* 互動狀態 */
.neon-circle:hover:not([disabled])  { transform: scale(1.08); }
.neon-circle:active:not([disabled]) { transform: scale(0.94); }

.neon-circle[disabled] {
  opacity: 0.4;
  cursor:  not-allowed;
  box-shadow: none;
  border-color: #666;
  color: #666;
}

.label{
    color:#51cef8;
}

/* 霓虹文字閃爍 */
@keyframes flicker {
  0%, 18%, 22%, 25%, 53%, 57%, 100% {
    opacity: 1;
  }
  20%, 24%, 55% {
    opacity: 0.6;
  }
}
.neon-circle span { 
    animation: flicker 3s infinite; }
</style>