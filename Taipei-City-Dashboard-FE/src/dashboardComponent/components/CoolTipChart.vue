<script setup>
import { ref, computed, defineProps, defineEmits, useAttrs } from 'vue'

/* ─── Props & Emits ─── */
const props = defineProps({
  label:    { type: String, default: '酷知識' },
  size:     { type: Number, default: 150 },      // 直徑
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
    v-if="activeChart === 'CoolTipChart'"
  >
    <div>
      <br/>
      <center>
      <div
        v-if="series.length"
        :key="series[0].name"
      >
        <div
          style="color: #FFFFFF"
        >
          {{ series[0].name }}
        </div>
      </div>
    </center>
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

.CoolTipChart {
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

@keyframes breathe{
  0%,100%{
    box-shadow:
    0 0 10px  #259cc3,
    0 0 20px  #259cc3,
    0 0 40px  #259cc3,
    /* 0 0 40px  #259cc3, */
    0 0 60px  #222327,
    /* 0 0 80px  #259cc3, */
    inset 0 0 10px #259cc3,
    inset 0 0 20px #259cc3;
    transform: scale(1);          /* 大小正常 */
  }
  50%{
    box-shadow:
    0 0 5px  #259cc3,
    0 0 10px  #259cc3,
    0 0 20px  #259cc3,
    /* 0 0 40px  #259cc3, */
    0 0 30px  #222327,
    /* 0 0 80px  #259cc3, */
    inset 0 0 5px #259cc3,
    inset 0 0 10px #259cc3;
    transform: scale(0.96);       /* 微微縮小，營造吸氣感 */
  }
}
.neon-circle{
  /* 保留原有屬性… */
  animation: breathe 3s ease-in-out infinite;
}

/* ────────── 4. Hover / Active 時仍可額外縮放 ────────── */
.neon-circle:hover:not([disabled]){
  animation: breathe 1s ease-in-out infinite;
}
.neon-circle:active:not([disabled]){ transform: scale(2); }

@keyframes shake{
  0%,100%{ transform:translateX(0); }
  /* 向左偏移 */
  10%,30%,50%,70%,90%{ transform:translateX(-4px); }
  /* 向右偏移 */
  20%,40%,60%,80%{ transform:translateX(4px); }
}

.dialog-btn{
  margin:0 auto;display:block;padding:6px 22px;border:none;border-radius:20px;
  background:#259cc3;color:#FFF;font-weight:600;cursor:pointer;

  /* 原本就有的滑鼠縮放動畫 */
  transition:transform .15s ease-in-out;

  /* ★ 新增：持續搖動 ★ */
  animation:shake .8s infinite;
}

/* 滑鼠 hover/active 時，先套用 scale，再繼續疊加搖動 */
.dialog-btn:hover { transform:scale(1.05); }
.dialog-btn:active{ transform:scale(.95);  }

</style>