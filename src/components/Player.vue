<template>
    <APlayer v-if="playList[0]"
             ref="player"
             :audio="playList"
             :autoplay="store.playerAutoplay"
             :theme="theme"
             :autoSwitch="false"
             :loop="store.playerLoop"
             :order="store.playerOrder"
             :volume="volume"
             :showLrc="true"
             :listFolded="listFolded"
             :listMaxHeight="listMaxHeight"
             :noticeSwitch="false"
             @play="onPlay"
             @pause="onPause"
             @timeupdate="onTimeUp"
             @error="loadMusicError" />
</template>

<script setup>
    import { MusicOne, PlayWrong } from "@icon-park/vue-next";
    import { mainStore } from "@/store";
    import APlayer from "@worstone/vue-aplayer";
    import { nextTick } from "vue";

    const store = mainStore();

    // 获取播放器 DOM
    const player = ref(null);

    // 歌曲播放列表
    const playList = ref([]);

    // 歌曲播放项
    const playIndex = ref(0);

    // 配置项
    const props = defineProps({
        theme: {
            type: String,
            default: "#efefef",
        },
        volume: {
            type: Number,
            default: 0.7,
            validator: (value) => {
                return value >= 0 && value <= 1;
            },
        },
        songServer: {
            type: String,
            default: "netease",
        },
        songType: {
            type: String,
            default: "playlist",
        },
        songId: {
            type: String,
            default: "7452421335",
        },
        listFolded: {
            type: Boolean,
            default: false,
        },
        listMaxHeight: {
            type: Number,
            default: 420,
        },
    });

    const listHeight = computed(() => {
        return props.listMaxHeight + "px";
    });

    // 初始化播放器
    onMounted(async () => {
        // 确保 store.playerAutoplay 为 true
        store.playerAutoplay = true;

        // 设置本地 MP3 文件的播放列表
        playList.value = [
            {
                name: "Song 1",
                artist: "Artist 1",
                url: "/music/song1.mp3",
                cover: "",
                lrc: "",
            },
            {
                name: "Song 2",
                artist: "Artist 2",
                url: "/music/song2.mp3",
                cover: "",
                lrc: "",
            },
        ];

        // 更改播放器加载状态
        store.musicIsOk = true;
        console.log("音乐加载完成");
        console.log("playList:", playList.value);
        console.log("playIndex:", playIndex.value);
        console.log("volume:", props.volume);
        console.log("autoplay:", store.playerAutoplay);

        // 确保播放器初始化后再尝试播放
        await nextTick();
        if (player.value && store.playerAutoplay) {
            console.log("尝试自动播放...");
            player.value.play();
        }
    });

    // 播放
    const onPlay = () => {
        console.log("播放");
        playIndex.value = player.value.aplayer.index;
        // 播放状态
        store.setPlayerState(player.value.audioRef.paused);
        // 储存播放器信息
        store.setPlayerData(playList.value[playIndex.value].name, playList.value[playIndex.value].artist);
        ElMessage({
            message: store.getPlayerData.name + " - " + store.getPlayerData.artist,
            grouping: true,
            icon: h(MusicOne, {
                theme: "filled",
                fill: "#efefef",
            }),
        });
    };

    // 暂停
    const onPause = () => {
        console.log("暂停");
        store.setPlayerState(player.value.audioRef.paused);
    };

    // 音频时间更新事件
    const onTimeUp = () => {
        let lyrics = player.value.aplayer.lyrics[playIndex.value];
        let lyricIndex = player.value.aplayer.lyricIndex;
        if (!lyrics || !lyrics[lyricIndex]) {
            return;
        }
        let lrc = lyrics[lyricIndex][1];
        if (lrc === "Loading") {
            lrc = "歌词加载中";
        } else if (lrc === "Not available") {
            lrc = "歌词加载失败";
        }
        store.setPlayerLrc(lrc);
    };

    // 切换播放暂停事件
    const playToggle = () => {
        player.value.toggle();
    };

    // 切换音量事件
    const changeVolume = (value) => {
        player.value.setVolume(value, false);
    };

    // 切换上下曲
    const changeSong = (type) => {
        type === 0 ? player.value.skipBack() : player.value.skipForward();
        nextTick(() => {
            player.value.play();
        });
    };

    // 切换歌曲列表状态
    const toggleList = () => {
        player.value.toggleList();
    };

    // 加载音频错误
    const loadMusicError = () => {
        let notice = "";
        if (playList.value.length > 1) {
            notice = "播放歌曲出现错误，播放器将在 2s 后进行下一首";
        } else {
            notice = "播放歌曲出现错误";
        }
        ElMessage({
            message: notice,
            grouping: true,
            icon: h(PlayWrong, {
                theme: "filled",
                fill: "#EFEFEF",
                duration: 2000,
            }),
        });
        console.error(
            "播放歌曲: " + player.value.aplayer.audio[player.value.aplayer.index].name + " 出现错误",
        );
    };

    // 暴露子组件方法
    defineExpose({ playToggle, changeVolume, changeSong, toggleList });
</script>

<style lang="scss" scoped>
    .aplayer {
        width: 80%;
        border-radius: 6px;
        font-family: "HarmonyOS_Regular", sans-serif !important;

        :deep(.aplayer-body) {
            background-color: transparent;

            .aplayer-pic {
                display: none;
            }

            .aplayer-info {
                margin-left: 0;
                background-color: #ffffff40;
                border-color: transparent !important;

                .aplayer-music {
                    flex-grow: initial;
                    margin-bottom: 2px;
                    overflow: initial;

                    .aplayer-title {
                        font-size: 16px;
                        margin-right: 6px;
                    }

                    .aplayer-author {
                        color: #efefef;
                    }
                }

                .aplayer-lrc {
                    text-align: left;
                    margin: 7px 0 6px 6px;
                    height: 44px;
                    mask: linear-gradient( #fff 15%, #fff 85%, hsla(0deg, 0%, 100%, 0.6) 90%, hsla(0deg, 0%, 100%, 0) );
                    -webkit-mask: linear-gradient( #fff 15%, #fff 85%, hsla(0deg, 0%, 100%, 0.6) 90%, hsla(0deg, 0%, 100%, 0) );

                    &::before,
                    &::after {
                        display: none;
                    }

                    p {
                        color: #efefef;
                    }

                    .aplayer-lrc-current {
                        font-size: 0.95rem;
                        margin-bottom: 4px !important;
                    }
                }

                .aplayer-controller {
                    display: none;
                }
            }
        }

        :deep(.aplayer-list) {
            margin-top: 6px;
            height: v-bind(listHeight);
            background-color: transparent;

            ol {
                &::-webkit-scrollbar-track {
                    background-color: transparent;
                }

                li {
                    border-color: transparent;

                    &.aplayer-list-light {
                        background: #ffffff40;
                        border-radius: 6px;
                    }

                    &:hover {
                        background: #ffffff26 !important;
                        border-radius: 6px !important;
                    }

                    .aplayer-list-index,
                    .aplayer-list-author {
                        color: #efefef;
                    }
                }
            }
        }
    }
</style>