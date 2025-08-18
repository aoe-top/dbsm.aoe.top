<template>
    <div class="qa">
        <div v-if="!started" class="start">
            <p>开始前请确认：您是成年（18+）且自愿参与本测试。</p>
            <label><input type="checkbox" v-model="consent" /> 我确认并同意</label>
            <div class="actions">
                <button :disabled="!consent" @click="start">开始测试</button>
            </div>
        </div>

        <div v-else-if="!finished" class="questions">
            <div class="progress-bar" aria-hidden="true">
                <div class="progress-fill" :style="{ width: ((current) / (questions.length - 1)) * 100 + '%' }"></div>
            </div>
            <div class="meta"
                style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px;color:var(--muted)">
                <div>问题 {{ current + 1 }} / {{ questions.length }}</div>
                <div>答题提示：选择 1（非常不同意）到 5（非常同意）</div>
            </div>
            <h3 class="qtext">{{ questions[current].text }}</h3>

            <div class="options">
                <label v-for="n in 5" :key="n" :class="['opt', answers[current] === n ? 'selected' : '']"
                    @click="select(n)">
                    <input type="radio" :name="questions[current].id" :value="n" v-model.number="answers[current]" />
                    <span>{{ n }} - {{ optionLabel(n) }}</span>
                </label>
            </div>

            <div class="nav">
                <button class="btn secondary" @click="prev" :disabled="current === 0">上一步</button>
                <div>
                    <button class="btn" @click="next" :disabled="!answers[current]">{{ current + 1 === questions.length
                        ?
                        '提交' : '下一题' }}</button>
                </div>
            </div>
        </div>

        <div v-else class="result">
            <h2>测试结果</h2>
            <p>你的倾向得分（越高表示越倾向该分类）：</p>
            <ul class="result-list">
                <li v-for="(v, k) in totals" :key="k" class="result-item">
                    <div style="width:140px">{{ formatLabel(k) }}</div>
                    <div class="result-bar">
                        <div class="result-fill" :style="{ width: Math.min(100, Math.abs(v) * 12) + '%' }"></div>
                    </div>
                    <div style="width:60px;text-align:right">{{ v.toFixed(2) }}</div>
                </li>
            </ul>
            <p class="interpret">主要倾向： <strong>{{ topLabel }}</strong></p>
            <div style="margin-top:12px;display:flex;gap:8px">
                <button class="btn" @click="restart">重新测试</button>
                <button class="btn secondary" @click="exportResults">导出结果</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import questions from '../data/questions'

// 响应式数据
const consent = ref(false)
const started = ref(false)
const finished = ref(false)
const current = ref(0)
const answers = ref(Array(questions.length).fill(null))
const totals = ref({})
const autoAdvanceTimer = ref(null)

// 计算属性
const topLabel = computed(() => {
    if (!totals.value) return ''
    const entries = Object.entries(totals.value)
    entries.sort((a, b) => b[1] - a[1])
    const top = entries[0]
    return formatLabel(top[0])
})

// 方法定义
const start = () => {
    started.value = true
}

const optionLabel = (n) => {
    const map = { 1: '非常不同意', 2: '不同意', 3: '中立', 4: '同意', 5: '非常同意' }
    return map[n]
}

const select = (n) => {
    // 设置答案
    answers.value.splice(current.value, 1, n)

    try {
        localStorage.setItem('dbsm_last_answers', JSON.stringify(answers.value))
    } catch (e) { }

    // 清除任何待处理的自动前进
    if (autoAdvanceTimer.value) {
        clearTimeout(autoAdvanceTimer.value)
        autoAdvanceTimer.value = null
    }

    // 小延迟以允许UI高亮，然后自动前进或提交
    autoAdvanceTimer.value = setTimeout(() => {
        autoAdvanceTimer.value = null
        if (current.value + 1 === questions.length) {
            calculate()
        } else {
            current.value++
        }
    }, 300)
}

const prev = () => {
    if (autoAdvanceTimer.value) {
        clearTimeout(autoAdvanceTimer.value)
        autoAdvanceTimer.value = null
    }
    if (current.value > 0) current.value--
}

const next = () => {
    if (!answers.value[current.value]) return
    if (current.value + 1 === questions.length) {
        calculate()
    } else {
        current.value++
    }
}

const calculate = () => {
    const result = { dominant: 0, submissive: 0, switcher: 0, sensation: 0 }
    questions.forEach((q, i) => {
        const v = answers.value[i] || 3
        Object.keys(q.weights).forEach(k => {
            result[k] += q.weights[k] * (v - 3)
        })
    })
    totals.value = result
    finished.value = true
}

const formatLabel = (k) => {
    const map = {
        dominant: '主导 (Dominant)',
        submissive: '臣服 (Submissive)',
        switcher: '双向 (Switch)',
        sensation: '感官寻求 (Sensation)'
    }
    return map[k] || k
}

const restart = () => {
    if (autoAdvanceTimer.value) {
        clearTimeout(autoAdvanceTimer.value)
        autoAdvanceTimer.value = null
    }
    consent.value = false
    started.value = false
    finished.value = false
    current.value = 0
    answers.value = Array(questions.length).fill(null)
    totals.value = {}
    try {
        localStorage.removeItem('dbsm_last_answers')
    } catch (e) { }
}

const exportResults = () => {
    const payload = { answers: answers.value, totals: totals.value, date: new Date().toISOString() }
    const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' })
    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = 'dbsm_results.json'
    document.body.appendChild(a)
    a.click()
    a.remove()
    URL.revokeObjectURL(url)
}

// 生命周期钩子
onMounted(() => {
    try {
        const raw = localStorage.getItem('dbsm_last_answers')
        if (raw) {
            const parsed = JSON.parse(raw)
            if (Array.isArray(parsed) && parsed.length === questions.length) {
                answers.value = parsed
            }
        }
    } catch (e) { }
})
</script>

<style scoped>
.qa {
    border: 1px solid #eee;
    padding: 18px;
    border-radius: 8px
}

.start p {
    margin: 0 0 8px
}

.actions {
    margin-top: 12px
}

.actions button {
    padding: 8px 14px
}

.progress {
    color: #666;
    margin-bottom: 8px
}

.qtext {
    margin: 8px 0
}

.options {
    display: flex;
    flex-direction: column
}

.opt {
    margin: 6px 0
}

.nav {
    margin-top: 12px
}

.nav button {
    margin-right: 8px;
    padding: 8px 12px
}

.result ul {
    list-style: none;
    padding: 0
}

.interpret {
    margin-top: 12px
}
</style>