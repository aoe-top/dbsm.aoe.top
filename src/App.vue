<template>
    <div class="container-card">
        <header>
            <h1>BDSM 倾向人格测试</h1>
            <p class="subtitle">
                非诊断性工具，仅用于自我探索与了解偏好。所有活动必须基于成年与双方自愿、知情、及安全原则。
            </p>
        </header>

        <Questionnaire />

        <footer>
            <small>由本地应用生成 — 保持隐私，不会自动上报结果。</small>
            <div class="friendly-links-panel">
                <p class="friendly-links-title">友情链接</p>
                <div v-if="pending" class="friendly-links-state">
                    友链加载中...
                </div>
                <div v-else-if="failed" class="friendly-links-state">
                    友链加载失败，请稍后重试。
                </div>
                <div v-else class="friendly-links-list">
                    <a
                        v-for="link in friendLinks"
                        :key="link.url"
                        :href="link.url"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="friendly-links-item">
                        {{ link.name }}
                    </a>
                </div>
            </div>
        </footer>
    </div>
</template>

<script setup>
import Questionnaire from "./components/Questionnaire.vue";
import { onMounted, ref } from "vue";

const friendLinks = ref([]);
const pending = ref(true);
const failed = ref(false);

const loadFriendLinks = async () => {
    try {
        const response = await fetch("https://api.aoe.top/api/friendly/links");
        if (!response.ok) {
            throw new Error(`Failed to load links: ${response.status}`);
        }

        const data = await response.json();
        friendLinks.value = Array.isArray(data) ? data : [];
    } catch (error) {
        console.error(error);
        failed.value = true;
    } finally {
        pending.value = false;
    }
};

onMounted(loadFriendLinks);
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
    color: #666;
}

.friendly-links-panel {
    margin-top: 14px;
    padding-top: 8px;
}

.friendly-links-title {
    margin: 0;
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: #4b5563;
}

.friendly-links-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.3rem 0.5rem;
    margin-top: 0.45rem;
}

.friendly-links-item,
.friendly-links-state {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-height: 0;
    padding: 0.2rem 0.4rem;
    border-radius: 999px;
    border: 0;
    background: transparent;
    color: #4b5563;
    font-size: 12px;
    line-height: 1.4;
    text-decoration: none;
    transition:
        color 0.2s ease,
        opacity 0.2s ease;
}

.friendly-links-item:hover {
    color: #111827;
}

.friendly-links-state {
    width: fit-content;
    margin: 0.45rem auto 0;
}
</style>
