<template>
    <div class="container-card">
        <header>
            <h1>BDSM 倾向人格测试</h1>
            <p class="subtitle">非诊断性工具，仅用于自我探索与了解偏好。所有活动必须基于成年与双方自愿、知情、及安全原则。</p>
        </header>

        <Questionnaire />

        <footer>
            <small>由本地应用生成 — 保持隐私，不会自动上报结果。</small>
        </footer>
    </div>
</template>

<script setup>
import Questionnaire from './components/Questionnaire.vue'
</script>

<style scoped>
.container {
    max-width: 760px;
    margin: 28px auto;
    padding: 16px;
    font-family: Arial, Helvetica, sans-serif;
    color: #111;
}

header h1 {
    margin: 0 0 8px 0;
}

.subtitle {
    margin: 0 0 20px 0;
    color: #555;
}

footer {
    text-align: center;
    margin-top: 24px;
    color: #666
}
</style>
