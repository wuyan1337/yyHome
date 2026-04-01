<template>
    <div id="loader-wrapper" :class="{ loaded: isLoaded }">
        <div class="loader">
            <div class="dynamic-bg" />
            <div class="particle-bg">
                <span v-for="i in 30" :key="i" class="particle" />
            </div>
            <div class="progress-container">
                <div class="progress-bar">
                    <div class="progress-fill" :class="{ shake: progress >= 80 && progress < 85 }" />
                    <div class="progress-glow" />
                </div>
                <div class="progress-text">
                    <span class="site-name">{{ siteName }}</span>
                    <span class="loading-tip">Loading... {{ Math.round(progress) }}%</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { ref, onMounted, watch } from "vue";
    import { mainStore } from "@/store";

    const store = mainStore();
    const siteName = import.meta.env.VITE_SITE_NAME;
    const progress = ref(0);
    const isLoaded = ref(false);

    onMounted(() => {
        let startTime = performance.now();
        const durationFast = 1500;
        const durationSlow = 1200;
        const totalDuration = durationFast + durationSlow;

        const updateProgress = (currentTime) => {
            const elapsed = currentTime - startTime;
            let t = Math.min(elapsed / totalDuration, 1);

            if (t < durationFast / totalDuration) {
                const tFast = elapsed / durationFast;
                progress.value = 80 * easeOutCubic(tFast);
            } else {
                const tSlow = (elapsed - durationFast) / durationSlow;
                progress.value = 80 + 20 * easeOutCubic(tSlow);
            }

            progress.value = Math.min(Math.max(progress.value, 0), 100);
            if (progress.value < 100) {
                requestAnimationFrame(updateProgress);
            }
        };

        requestAnimationFrame(updateProgress);
    });

    watch(progress, (newValue) => {
        if (newValue >= 100) {
            isLoaded.value = true;
            store.imgLoadStatus = true;
        }
    });

    const easeOutCubic = (t) => {
        return 1 - Math.pow(1 - t, 3);
    };
</script>

<style scoped lang="scss">
    #loader-wrapper {
        position: fixed;
        inset: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        background: #0d0d1a;
        z-index: 1000;
        transition: opacity 1s ease, visibility 1s ease;
        overflow: hidden;

        &.loaded {
            opacity: 0;
            visibility: hidden;
        }

        .loader {
            position: relative;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;

            .dynamic-bg {
                position: absolute;
                width: 100%;
                height: 100%;
                background: radial-gradient(circle at center, #1e1e2f, #0d0d1a);
                animation: pulseBg 4s infinite alternate ease-in-out;
            }

            .particle-bg {
                position: absolute;
                inset: 0;
                overflow: hidden;

                .particle {
                    position: absolute;
                    width: 6px;
                    height: 6px;
                    background: rgba(255, 255, 255, 0.5);
                    border-radius: 50%;
                    animation: floatParticle 6s infinite ease-in-out;
                    opacity: 0.5;
                    box-shadow: 0 0 12px rgba(255, 255, 255, 0.4);

                    @for $i from 1 through 30 {
                        &:nth-child(#{$i}) {
                            left: random(100) * 1%;
                            top: random(100) * 1%;
                            animation-delay: random(3000) * -0.01s;
                            transform: scale(random(5) * 0.3);
                        }
                    }
                }
            }

            .progress-container {
                width: 80%;
                max-width: 600px;
                z-index: 10;
                text-align: center;

                .progress-bar {
                    position: relative;
                    height: 18px;
                    background: rgba(255, 255, 255, 0.1);
                    border-radius: 999px;
                    overflow: hidden;
                    box-shadow: 0 0 25px rgba(255, 0, 128, 0.4);

                    .progress-fill {
                        height: 100%;
                        width: v-bind("progress + '%' ");
                        background: linear-gradient(270deg, #ff416c, #ff4b2b, #ffe600);
                        background-size: 300% 300%;
                        border-radius: 999px;
                        animation: shimmer 1.5s linear infinite;
                        transition: width 0.2s ease-out;

                        &.shake {
                            animation: shake 0.3s ease-in-out;
                        }
                    }

                    .progress-glow {
                        position: absolute;
                        top: -10px;
                        left: 0;
                        width: 100%;
                        height: 100%;
                        background: radial-gradient(circle, rgba(255, 64, 129, 0.3), transparent);
                        filter: blur(15px);
                        animation: glow 2.5s ease-in-out infinite;
                    }
                }

                .progress-text {
                    margin-top: 30px;
                    color: #fff;

                    .site-name {
                        font-size: 32px;
                        font-weight: bold;
                        letter-spacing: 2px;
                        animation: breathe 2s ease-in-out infinite;
                        text-shadow: 0 0 20px #ff4b2b;
                    }

                    .loading-tip {
                        font-size: 18px;
                        opacity: 0.8;
                        margin-top: 10px;
                    }
                }
            }
        }
    }

    @keyframes pulseBg {
        0% {
            background: radial-gradient(circle at center, #1e1e2f, #0d0d1a);
        }

        100% {
            background: radial-gradient(circle at center, #2a2a4a, #0d0d1a);
        }
    }

    @keyframes floatParticle {
        0%, 100% {
            transform: translateY(0);
        }

        50% {
            transform: translateY(-20px);
        }
    }

    @keyframes shimmer {
        0% {
            background-position: 0% 50%;
        }

        100% {
            background-position: 100% 50%;
        }
    }

    @keyframes breathe {
        0%, 100% {
            text-shadow: 0 0 10px #ff4b2b;
        }

        50% {
            text-shadow: 0 0 30px #ffe600;
        }
    }

    @keyframes glow {
        0% {
            transform: scaleX(0.9);
        }

        50% {
            transform: scaleX(1.1);
        }

        100% {
            transform: scaleX(0.9);
        }
    }

    @keyframes shake {
        0% {
            transform: translateX(0);
        }

        25% {
            transform: translateX(-2px);
        }

        50% {
            transform: translateX(2px);
        }

        75% {
            transform: translateX(-1px);
        }

        100% {
            transform: translateX(0);
        }
    }
</style>
