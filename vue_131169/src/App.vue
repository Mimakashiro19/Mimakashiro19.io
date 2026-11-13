<script setup lang="ts">
import { ref } from 'vue'
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
import TestPage from './components/TestPage.vue'

const currentPage = ref<'home' | 'about' | 'calculator' | 'test'>('home')

const profile = ref({
  name: 'น.ส.ณัฐจิราภา รุ่งเรือง',
  nickname: 'ซาลาเปา',
  role: 'IT 3',
  bio: 'เดียวก็จะจบแล้ว จะหนีไปเรียนนิเทศแล้ว'
})

// --- ระบบเครื่องคิดเลข ---
const display = ref('0')
const expression = ref('') // เก็บข้อความแสดงทุกตัวที่กดและเครื่องหมาย
const prevValue = ref<number | null>(null)
const operator = ref<string | null>(null)
const waitingForNewValue = ref(false)
const isCalculated = ref(false) // เช็คว่าเพิ่งกด = ไปหรือไม่

// ระบบจัดการสีปุ่มเวลากด
const activeKey = ref<string | null>(null)
let flashTimeout: ReturnType<typeof setTimeout> | null = null

const triggerFlash = (keyId: string) => {
  activeKey.value = keyId
  if (flashTimeout) clearTimeout(flashTimeout)
  flashTimeout = setTimeout(() => {
    activeKey.value = null
  }, 180) // สีจะค้างไว้ 0.18 วินาทีแล้วหายไป
}

// แปลงเครื่องหมายโค้ดให้เป็นสัญลักษณ์สวยๆ บนหน้าจอ
const formatOpSymbol = (op: string) => {
  const symbols: Record<string, string> = {
    '/': '÷',
    '*': '×',
    '-': '−',
    '+': '+'
  }
  return symbols[op] || op
}

const inputDigit = (digit: string) => {
  triggerFlash(digit)

  // ถ้าเพิ่งกด = เสร็จ แล้วกดตัวเลขใหม่ ให้เริ่มสมการใหม่ทั้งหมด
  if (isCalculated.value) {
    display.value = digit
    expression.value = digit
    isCalculated.value = false
    waitingForNewValue.value = false
    return
  }

  if (waitingForNewValue.value) {
    display.value = digit
    expression.value += digit
    waitingForNewValue.value = false
  } else {
    if (display.value === '0' || display.value === 'Error') {
      display.value = digit
      // ป้องกันเลข 0 นำหน้าซ้ำซ้อนใน expression
      if (expression.value === '' || expression.value === '0' || expression.value === 'Error') {
        expression.value = digit
      } else {
        expression.value = expression.value.slice(0, -1) + digit
      }
    } else {
      display.value += digit
      expression.value += digit
    }
  }
}

const inputDecimal = () => {
  triggerFlash('.')

  if (isCalculated.value) {
    display.value = '0.'
    expression.value = '0.'
    isCalculated.value = false
    waitingForNewValue.value = false
    return
  }

  if (waitingForNewValue.value) {
    display.value = '0.'
    expression.value += '0.'
    waitingForNewValue.value = false
    return
  }

  if (!display.value.includes('.')) {
    display.value += '.'
    expression.value += expression.value === '' ? '0.' : '.'
  }
}

const clearAll = () => {
  triggerFlash('C')
  display.value = '0'
  expression.value = ''
  prevValue.value = null
  operator.value = null
  waitingForNewValue.value = false
  isCalculated.value = false
}

const compute = (a: number, b: number, op: string): number | string => {
  switch (op) {
    case '+': return a + b
    case '-': return a - b
    case '*': return a * b
    case '/': return b === 0 ? 'Error' : Number((a / b).toFixed(8))
    default: return b
  }
}

const chooseOperator = (nextOp: string) => {
  triggerFlash(nextOp)
  const inputValue = parseFloat(display.value)
  if (isNaN(inputValue)) return

  const symbol = formatOpSymbol(nextOp)

  // ถ้าเพิ่งกด = แล้วกดเครื่องหมายต่อ ให้เอาผลลัพธ์มาคำนวณต่อเลย
  if (isCalculated.value) {
    prevValue.value = inputValue
    operator.value = nextOp
    expression.value = `${display.value} ${symbol} `
    waitingForNewValue.value = true
    isCalculated.value = false
    return
  }

  // ถ้ากดเครื่องหมายซ้ำติดกัน (เปลี่ยนใจจาก + เป็น -) ให้เปลี่ยนสัญลักษณ์ตัวท้ายสุด
  if (waitingForNewValue.value && operator.value !== null) {
    operator.value = nextOp
    expression.value = expression.value.replace(/\s[÷×−+]\s$/, ` ${symbol} `)
    return
  }

  if (prevValue.value === null) {
    prevValue.value = inputValue
    if (expression.value === '') expression.value = String(inputValue)
  } else if (operator.value && !waitingForNewValue.value) {
    const result = compute(prevValue.value, inputValue, operator.value)
    display.value = String(result)
    prevValue.value = typeof result === 'number' ? result : null
  }

  expression.value += ` ${symbol} `
  waitingForNewValue.value = true
  operator.value = nextOp
}

const calculateResult = () => {
  triggerFlash('=')
  if (operator.value === null || prevValue.value === null || waitingForNewValue.value) return
  
  const inputValue = parseFloat(display.value)
  const result = compute(prevValue.value, inputValue, operator.value)

  expression.value += ' ='
  display.value = String(result)
  prevValue.value = null
  operator.value = null
  waitingForNewValue.value = true
  isCalculated.value = true
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
      <nav class="menu">
        <button 
          :class="{ active: currentPage === 'home' }" 
          @click="currentPage = 'home'"
        >
          หน้าหลัก
        </button>
        <button 
          :class="{ active: currentPage === 'about' }" 
          @click="currentPage = 'about'"
        >
          แนะนำตัว
        </button>
        <button 
          :class="{ active: currentPage === 'calculator' }" 
          @click="currentPage = 'calculator'"
        >
          เครื่องคิดเลข
        </button>
        <button 
          :class="{ active: currentPage === 'test' }" 
          @click="currentPage = 'test'"
        >
          หน้าทดสอบใหม่
        </button>
      </nav>
    </div>
  </header>

  <main>
    <TheWelcome v-if="currentPage === 'home'" />

    <section v-else-if="currentPage === 'about'" class="about-page">
      <h2>แนะนำตัว</h2>
      <p><strong>ชื่อ:</strong> {{ profile.name }} ({{ profile.nickname }})</p>
      <p><strong>แผนก:</strong> {{ profile.role }}</p>
      <p class="bio-text">{{ profile.bio }}</p>
    </section>

    <section v-else-if="currentPage === 'calculator'" class="calc-page">
      <h2>เครื่องคิดเลข</h2>
      <div class="calculator">
        <!-- หน้าจอแสดงผลแบบ 2 บรรทัด -->
        <div class="calc-display">
          <div class="calc-history">{{ expression || '\u00A0' }}</div>
          <div class="calc-current">{{ display }}</div>
        </div>

        <div class="calc-keys">
          <button 
            class="key-clear" 
            :class="{ 'is-flashing': activeKey === 'C' }" 
            @click="clearAll"
          >C</button>
          <button 
            class="key-op" 
            :class="{ 'is-flashing': activeKey === '/' }" 
            @click="chooseOperator('/')"
          >÷</button>
          <button 
            class="key-op" 
            :class="{ 'is-flashing': activeKey === '*' }" 
            @click="chooseOperator('*')"
          >×</button>
          <button 
            class="key-op" 
            :class="{ 'is-flashing': activeKey === '-' }" 
            @click="chooseOperator('-')"
          >−</button>

          <button :class="{ 'is-flashing': activeKey === '7' }" @click="inputDigit('7')">7</button>
          <button :class="{ 'is-flashing': activeKey === '8' }" @click="inputDigit('8')">8</button>
          <button :class="{ 'is-flashing': activeKey === '9' }" @click="inputDigit('9')">9</button>
          <button 
            class="key-op" 
            :class="{ 'is-flashing': activeKey === '+' }" 
            @click="chooseOperator('+')"
          >+</button>

          <button :class="{ 'is-flashing': activeKey === '4' }" @click="inputDigit('4')">4</button>
          <button :class="{ 'is-flashing': activeKey === '5' }" @click="inputDigit('5')">5</button>
          <button :class="{ 'is-flashing': activeKey === '6' }" @click="inputDigit('6')">6</button>
          <button 
            class="key-eq" 
            :class="{ 'is-flashing': activeKey === '=' }" 
            @click="calculateResult"
          >=</button>

          <button :class="{ 'is-flashing': activeKey === '1' }" @click="inputDigit('1')">1</button>
          <button :class="{ 'is-flashing': activeKey === '2' }" @click="inputDigit('2')">2</button>
          <button :class="{ 'is-flashing': activeKey === '3' }" @click="inputDigit('3')">3</button>
          <button :class="{ 'is-flashing': activeKey === '0' }" @click="inputDigit('0')">0</button>

          <button 
            class="key-dot" 
            :class="{ 'is-flashing': activeKey === '.' }" 
            @click="inputDecimal"
          >.</button>
        </div>
      </div>
    </section>

    <TestPage v-else />
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.menu {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1rem;
}

.menu button {
  padding: 0.5rem 1rem;
  border: 1px solid #42b883;
  background: transparent;
  color: inherit;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.2s;
}

.menu button:hover,
.menu button.active {
  background-color: #42b883;
  color: #ffffff;
}

.about-page,
.calc-page {
  padding: 1.5rem;
  border: 1px solid var(--color-border, #e5e7eb);
  border-radius: 10px;
  line-height: 1.8;
}

.about-page h2,
.calc-page h2 {
  color: #42b883;
  margin-bottom: 1rem;
  border-bottom: 2px solid #42b883;
  display: inline-block;
}

.bio-text {
  margin-top: 1rem;
}

/* สไตล์เครื่องคิดเลข */
.calculator {
  max-width: 280px;
  margin-top: 0.5rem;
  padding: 1rem;
  border-radius: 12px;
  background-color: var(--color-background-soft, #f8fafc);
  border: 1px solid var(--color-border, #cbd5e1);
}

.calc-display {
  background-color: #1e293b;
  color: #ffffff;
  font-family: monospace;
  text-align: right;
  padding: 0.65rem 1rem;
  border-radius: 8px;
  margin-bottom: 0.75rem;
  overflow-x: auto;
}

/* บรรทัดบนแสดงตัวเลขและเครื่องหมายทั้งหมดที่กด */
.calc-history {
  font-size: 0.9rem;
  color: #94a3b8;
  min-height: 1.25rem;
  white-space: nowrap;
}

/* บรรทัดล่างแสดงตัวเลขปัจจุบัน / ผลลัพธ์ */
.calc-current {
  font-size: 1.75rem;
  font-weight: bold;
  line-height: 1.2;
}

.calc-keys {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.5rem;
}

.calc-keys button {
  padding: 0.85rem;
  font-size: 1.1rem;
  font-weight: 600;
  border: 1px solid var(--color-border, #cbd5e1);
  border-radius: 8px;
  background-color: var(--color-background, #ffffff);
  color: inherit;
  cursor: pointer;
  transition: background-color 0.15s ease, transform 0.1s ease, color 0.15s ease;
}

.calc-keys button:hover {
  opacity: 0.9;
}

.calc-keys .key-op {
  background-color: #e2e8f0;
  color: #0f172a;
}

.calc-keys .key-clear {
  background-color: #ef4444;
  color: #ffffff;
  border-color: #ef4444;
}

.calc-keys .key-eq {
  background-color: #42b883;
  color: #ffffff;
  border-color: #42b883;
  grid-row: span 2;
}

.calc-keys .key-dot {
  grid-column: span 4;
}

/* สีที่จะสว่างวาบขึ้นมาแป๊บหนึ่งตอนกดปุ่ม */
.calc-keys button.is-flashing {
  background-color: #38bdf8 !important;
  color: #ffffff !important;
  border-color: #0284c7 !important;
  transform: scale(0.95);
}

.calc-keys .key-op.is-flashing {
  background-color: #f59e0b !important;
  border-color: #d97706 !important;
}

.calc-keys .key-clear.is-flashing {
  background-color: #fca5a5 !important;
  color: #7f1d1d !important;
}

.calc-keys .key-eq.is-flashing {
  background-color: #6ee7b7 !important;
  color: #064e3b !important;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>