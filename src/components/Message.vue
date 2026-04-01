<template>
    <!-- 基本信息 -->
    <div class="message">
        <!-- Logo -->
        <div class="logo">
            <img class="logo-img" :src="siteLogo" alt="logo" />
            <div :class="{ name: true, 'text-hidden': true, long: siteUrl[0].length >= 6 }">
                <span class="bg">{{ siteUrl[0] }}</span>
                <span class="sm">.{{ siteUrl[1] }}</span>
            </div>
        </div>
        <!-- 简介 -->
        <div class="description cards" @click="changeBox">
            <div class="content">
                <Icon size="16">
                    <QuoteLeft />
                </Icon>
                <Transition name="fade" mode="out-in">
                    <div :key="descriptionText.hello + descriptionText.text" class="text">
                        <p>{{ descriptionText.hello }}</p>
                        <p>{{ descriptionText.text }}</p>
                    </div>
                </Transition>
                <Icon size="16">
                    <QuoteRight />
                </Icon>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { Icon } from "@vicons/utils";
    import { QuoteLeft, QuoteRight } from "@vicons/fa";
    import { Error } from "@icon-park/vue-next";
    import { mainStore } from "@/store";
    const store = mainStore();
    import { computed, reactive, watch, h } from "vue";
    import { ElMessage } from "element-plus";

    // 主页站点 logo
    const siteLogo = import.meta.env.VITE_SITE_MAIN_LOGO;

    // 站点链接拆分
    const siteUrl = computed(() => {
        const url = import.meta.env.VITE_SITE_URL;
        if (!url) return "imsyy.top".split(".");
        if (url.startsWith("http://") || url.startsWith("https://")) {
            const urlFormat = url.replace(/^(https?:\/\/)/, "");
            return urlFormat.split(".");
        }
        return url.split(".");
    });

    // 文案列表（从 .env 中）
    const descriptions = [
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_1 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_2 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_3 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_4 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_5 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_6 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_7 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_8 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_9 },
        { hello: import.meta.env.VITE_DESC_HELLO, text: import.meta.env.VITE_DESC_TEXT_10 },
    ];

    // 当前文案（初始化随机）
    const descriptionText = reactive(
        descriptions[Math.floor(Math.random() * descriptions.length)]
    );

    // 点击切换盒子 & 随机文案
    const changeBox = () => {
        if (store.getInnerWidth >= 721) {
            store.boxOpenState = !store.boxOpenState;

            const random = Math.floor(Math.random() * descriptions.length);
            descriptionText.hello = descriptions[random].hello;
            descriptionText.text = descriptions[random].text;
        } else {
            ElMessage({
                message: "当前页面宽度不足以开启盒子",
                grouping: true,
                icon: h(Error, {
                    theme: "filled",
                    fill: "#efefef",
                }),
            });
        }
    };
</script>

<style lang="scss" scoped>
    .message {
        .logo {
            display: flex;
            flex-direction: row;
            align-items: center;
            animation: fade 0.5s;
            max-width: 460px;

            .logo-img {
                border-radius: 50%;
                width: 120px;
            }

            .name {
                width: 100%;
                padding-left: 22px;
                transform: translateY(-8px);
                font-family: "Pacifico-Regular";

                .bg {
                    font-size: 5rem;
                }

                .sm {
                    margin-left: 6px;
                    font-size: 2rem;

                    @media (min-width: 721px) and (max-width: 789px) {
                        display: none;
                    }
                }
            }

            @media (max-width: 768px) {
                .logo-img {
                    width: 100px;
                }

                .name {
                    height: 128px;

                    .bg {
                        font-size: 4.5rem;
                    }
                }
            }

            @media (max-width: 720px) {
                max-width: 100%;
            }
        }

        .description {
            padding: 1rem;
            margin-top: 3.5rem;
            max-width: 460px;
            animation: fade 0.5s;

            .content {
                display: flex;
                justify-content: space-between;

                .text {
                    margin: 0.75rem 1rem;
                    line-height: 2rem;
                    margin-right: auto;
                    transition: opacity 0.2s;

                    p {
                        &:nth-of-type(1) {
                            font-family: "Pacifico-Regular";
                        }
                    }
                }

                .xicon:nth-of-type(2) {
                    align-self: flex-end;
                }
            }

            @media (max-width: 720px) {
                max-width: 100%;
                pointer-events: none;
            }
        }
    }
</style>
