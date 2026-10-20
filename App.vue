<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// State
const display = ref('0')
const history = ref([])
const isEvaluated = ref(false)

// Fungsi untuk menambah karakter / angka / operator ke display
const append = (value) => {
  if (display.value === 'Error') clear()

  if (isEvaluated.value && !['+', '-', '*', '/', '%'].includes(value)) {
    display.value = value
    isEvaluated.value = false
    return
  }

  isEvaluated.value = false

  if (display.value === '0' && value !== '.') {
    display.value = value
    return
  }

  // Mencegah dua operator berdampingan
  const lastChar = display.value.slice(-1)
  const operators = ['+', '-', '*', '/', '%']
  if (operators.includes(lastChar) && operators.includes(value)) {
    display.value = display.value.slice(0, -1) + value
    return
  }

  display.value += value
}

// Clear All (AC)
const clear = () => {
  display.value = '0'
  isEvaluated.value = false
}

// Hapus 1 Karakter (Backspace)
const backspace = () => {
  if (display.value.length === 1 || display.value === 'Error') {
    display.value = '0'
  } else {
    display.value = display.value.slice(0, -1)
  }
}

// Fungsi Khusus Saintifik (Log, Sin, Cos, Tan, Sqrt, Pangkat, dll)
const applyScientific = (func) => {
  if (display.value === 'Error') return
  try {
    let currentVal = eval(display.value) // Evaluasi nilai saat ini terlebih dahulu
    let result = 0
    let label = ''

    switch (func) {
      case 'sqrt':
        result = Math.sqrt(currentVal)
        label = `√(${currentVal})`
        break
      case 'sqr':
        result = Math.pow(currentVal, 2)
        label = `(${currentVal})²`
        break
      case 'log':
        result = Math.log10(currentVal)
        label = `log(${currentVal})`
        break
      case 'ln':
        result = Math.log(currentVal)
        label = `ln(${currentVal})`
        break
      case 'sin':
        result = Math.sin(currentVal * (Math.PI / 180)) // Derajat ke Radian
        label = `sin(${currentVal}°)`
        break
      case 'cos':
        result = Math.cos(currentVal * (Math.PI / 180))
        label = `cos(${currentVal}°)`
        break
      case 'tan':
        result = Math.tan(currentVal * (Math.PI / 180))
        label = `tan(${currentVal}°)`
        break
      case 'pi':
        result = Math.PI
        label = 'π'
        break
      case 'e':
        result = Math.E
        label = 'e'
        break
    }

    const formattedResult = Number.isInteger(result) ? result.toString() : parseFloat(result.toFixed(6)).toString()
    history.value.unshift(`${label} = ${formattedResult}`)
    if (history.value.length > 5) history.value.pop()

    display.value = formattedResult
    isEvaluated.value = true
  } catch (error) {
    display.value = 'Error'
    isEvaluated.value = true
  }
}

// Hitung Hasil Utama (=)
const calculate = () => {
  if (display.value === 'Error') return

  try {
    const expression = display.value
    // Ganti simbol visual dengan operator JavaScript yang valid jika ada
    const sanitizedExpr = expression.replace(/×/g, '*').replace(/÷/g, '/')
    
    const result = Function(`'use strict'; return (${sanitizedExpr})`)()
    
    if (!isFinite(result) || isNaN(result)) {
      throw new Error('Invalid')
    }

    const formattedResult = Number.isInteger(result) ? result.toString() : parseFloat(result.toFixed(6)).toString()
    
    history.value.unshift(`${expression} = ${formattedResult}`)
    if (history.value.length > 5) history.value.pop()

    display.value = formattedResult
    isEvaluated.value = true
  } catch (error) {
    display.value = 'Error'
    isEvaluated.value = true
  }
}

// Menghapus Riwayat
const clearHistory = () => {
  history.value = []
}

// Dukungan Keyboard
const handleKeyDown = (e) => {
  if ((e.key >= '0' && e.key <= '9') || e.key === '.') {
    append(e.key)
  } else if (['+', '-', '*', '/','%'].includes(e.key)) {
    append(e.key)
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault()
    calculate()
  } else if (e.key === 'Backspace') {
    backspace()
  } else if (e.key === 'Escape') {
    clear()
  }
}

onMounted(() => window.addEventListener('keydown', handleKeyDown))
onUnmounted(() => window.removeEventListener('keydown', handleKeyDown))
</script>

<template>
  <div class="min-h-screen bg-slate-950 flex items-center justify-center p-4">
    <div class="w-full max-w-md bg-slate-900 rounded-3xl p-6 shadow-2xl border border-slate-800">
      
      <!-- Header / Brand -->
      <div class="flex justify-between items-center mb-3 text-slate-400 text-xs font-bold tracking-widest">
        <span>VUE SCIENTIFIC</span>
        <span class="text-cyan-400">PRO EDITION</span>
      </div>

      <!-- Display Layar -->
      <div class="bg-slate-950 rounded-2xl p-4 mb-4 text-right flex flex-col justify-end h-28 border border-slate-800/80 shadow-inner">
        <span class="text-slate-500 text-xs h-5 overflow-hidden font-mono">
          {{ history.length > 0 ? history[0] : '' }}
        </span>
        <span class="text-3xl font-extrabold text-cyan-400 tracking-tight overflow-x-auto whitespace-nowrap scrollbar-none font-mono">
          {{ display }}
        </span>
      </div>

      <!-- Grid Tombol Saintifik & Kalkulator -->
      <div class="grid grid-cols-4 gap-2.5 mb-4">
        <!-- Baris Tambahan Saintifik -->
        <button @click="applyScientific('sin')" class="sci-btn">sin</button>
        <button @click="applyScientific('cos')" class="sci-btn">cos</button>
        <button @click="applyScientific('tan')" class="sci-btn">tan</button>
        <button @click="applyScientific('sqrt')" class="sci-btn">√</button>

        <button @click="applyScientific('log')" class="sci-btn">log</button>
        <button @click="applyScientific('ln')" class="sci-btn">ln</button>
        <button @click="applyScientific('sqr')" class="sci-btn">x²</button>
        <button @click="append('%')" class="sci-btn">%</button>

        <button @click="applyScientific('pi')" class="sci-btn text-amber-400">π</button>
        <button @click="applyScientific('e')" class="sci-btn text-amber-400">e</button>
        <button @click="clear" class="sci-btn text-rose-400 bg-rose-500/10 hover:bg-rose-500/20">AC</button>
        <button @click="backspace" class="sci-btn text-slate-300 bg-slate-800 hover:bg-slate-700">⌫</button>

        <!-- Tombol Angka & Operator Standar -->
        <button @click="append('7')" class="calc-btn">7</button>
        <button @click="append('8')" class="calc-btn">8</button>
        <button @click="append('9')" class="calc-btn">9</button>
        <button @click="append('/')" class="op-btn">÷</button>

        <button @click="append('4')" class="calc-btn">4</button>
        <button @click="append('5')" class="calc-btn">5</button>
        <button @click="append('6')" class="calc-btn">6</button>
        <button @click="append('*')" class="op-btn">×</button>

        <button @click="append('1')" class="calc-btn">1</button>
        <button @click="append('2')" class="calc-btn">2</button>
        <button @click="append('3')" class="calc-btn">3</button>
        <button @click="append('-')" class="op-btn">-</button>

        <button @click="append('0')" class="calc-btn col-span-2">0</button>
        <button @click="append('.')" class="calc-btn">.</button>
        <button @click="append('+')" class="op-btn">+</button>

        <!-- Tombol Sama Dengan Lebar Penuh -->
        <button @click="calculate" class="col-span-4 bg-gradient-to-r from-cyan-600 to-indigo-600 text-white py-3.5 rounded-2xl font-bold text-lg shadow-lg hover:opacity-90 transition active:scale-[0.98]">
          =
        </button>
      </div>

      <!-- Panel Riwayat Hitungan -->
      <div v-if="history.length > 0" class="border-t border-slate-800 pt-3">
        <div class="flex justify-between items-center mb-1">
          <span class="text-[10px] font-semibold text-slate-500 uppercase tracking-wider">Riwayat</span>
          <button @click="clearHistory" class="text-[10px] text-rose-400 hover:underline">Hapus</button>
        </div>
        <ul class="space-y-1 max-h-20 overflow-y-auto">
          <li v-for="(item, idx) in history" :key="idx" class="text-xs text-slate-400 font-mono text-right">
            {{ item }}
          </li>
        </ul>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* Desain Tombol Kustom CSS Murni */
.calc-btn {
  background-color: rgb(30 41 59);
  color: rgb(241 245 249);
  padding: 0.75rem;
  border-radius: 1rem;
  font-weight: 600;
  font-size: 1rem;
  transition: all 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
}
.calc-btn:hover {
  background-color: rgb(51 65 85);
}
.calc-btn:active {
  transform: scale(0.95);
}

.op-btn {
  background-color: rgba(79, 70, 229, 0.2);
  color: rgb(129, 140, 248);
  padding: 0.75rem;
  border-radius: 1rem;
  font-weight: 700;
  font-size: 1.1rem;
  transition: all 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}
.op-btn:hover {
  background-color: rgba(79, 70, 229, 0.3);
}
.op-btn:active {
  transform: scale(0.95);
}

.sci-btn {
  background-color: rgb(15 23 42);
  color: rgb(148, 163, 184);
  padding: 0.6rem;
  border-radius: 0.75rem;
  font-weight: 600;
  font-size: 0.85rem;
  border: 1px solid rgb(30 41 59);
  transition: all 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}
.sci-btn:hover {
  background-color: rgb(30 41 59);
  color: rgb(255, 255, 255);
}
.sci-btn:active {
  transform: scale(0.95);
}
</style>