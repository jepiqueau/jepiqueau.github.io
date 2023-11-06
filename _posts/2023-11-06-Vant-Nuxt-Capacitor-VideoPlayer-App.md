# Vant Nuxt Capacitor Video Player Example Tutorial using Nuxt, Vant and the capacitor-video-player plugin
---

*last updated on November 6, 2023 by Quéau Jean Pierre*

In that tutorial we will learned how to create a Vant Nuxt application and implement the capacitor-video-player plugin to play video.

The application can be found at [vant-nuxt-videoplayer](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/vant-nuxt-videoplayer-app)

## Table of Contents

---
 - [Install New Nuxt Application](#install-new-nuxt-application)
 - [Change the name of the application](#change-the-name-of-the-application)
 - [Install Vant](#install-vant)
 - [Install Capacitor](#install-capacitor)
 - [Modify app.vue](#modify-appvue)
 - [Create Page Index and About](#create-page-index-and-about)
 - [Create Custom PageHeader Component](#create-custom-pageheader-component)
 - [Install capacitor-video-player plugin](#install-capacitor-video-player-plugin)
 - [Create Video Utilities](#create-video-utilities)
 - [Create Composables](#create-composables)
 - [Create FullscreenVideoPlayer Component](#create-fullscreenvideoplayer-component)
 - [Create Viewer Video Page](#create-viewer-video-page)
 - [Reference the VideoViewer Page in Index Page](#reference-the-videoviewer-page-in-index-page)
 - [Run the Application in the Web Browser](#run-the-application-in-the-web-browser)
 -[Run the Application in iOS Device](#run-the-application-in-ios-device)
 -[Run the Application in Android Device](#run-the-application-in-android-device)
 -[Conclusion](#conclusion)


### Install New Nuxt Application

 - Run the below command.

 ```bash
 npx nuxi@latest init vant-nuxt-videoplayer-app
 ```
 and select which package manager you would like to use. Below we assume that `npm` has been selected. If you have selected `pnpm` or `yarn` or `bun`, please adapt the following given commands.

### Change the name of the application

 - Open the `package.json` file and change the name.

 ```json
 {
 "name": "vant-nuxt-videoplayer-app"
 ...
 }
 ```

 - Open the `package-lock.json` file and change the name.

 ```json
 {
 "name": "vant-nuxt-videoplayer-app",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "vant-nuxt-videoplayer-app",
      ...
    }
    ...
  }
 }
 ```

### Install Vant

 - Run the following command.

 ```bash
 npm i --save-dev vant @vant/nuxt
 ```

 - open `nuxt.config.ts` file in your favorite file editor and modify it as shown.

 ```ts
    // https://nuxt.com/docs/api/configuration/nuxt-config
    export default defineNuxtConfig({
    devtools: { enabled: true },
    modules: ['@vant/nuxt'],
    })
 ```

### Install Capacitor

 - Run the following commands.

 ```bash
 npm i @capacitor/core
 npm i -D @capacitor/cli
 ```

 - Initialize your Capacitor config

 ```bash
 npx cap init
 ```
 and answer to the following questions.

 ```
 ? Name > YOUR_APP_NAME
 ? Package ID > YOUR_PACKAGE_ID
 ? Web asset directory > dist
 ```

 - Edit the `nuxt.config.ts`file to look like this.

 ```ts
    // https://nuxt.com/docs/api/configuration/nuxt-config
    export default defineNuxtConfig({
    app: {
        head: {
        title: 'Vant Nuxt VideoPlayer App',
        meta: [
            { name: 'viewport', content: 'viewport-fit=cover, width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no'},
            { name: 'format-detection', content: 'telephone=no' },
            { name: 'msapplication-tap-highlight', content:'no'},
            { name: 'apple-mobile-web-app-capable', content:'yes' },
            { name: 'apple-mobile-web-app-title', content:'Vant Nuxt VideoPlayer App' },
            { name: 'apple-mobile-web-app-status-bar-style', content:'black'},
        ]
        }
    },
    devtools: { enabled: true },
    modules: ['@vant/nuxt'],

    })
 ```

### Modify app.vue

 - open the file `app.vue` and replace the code by:

 ```html
    <template>
    <div>
        <NuxtPage />
    </div>
    </template>
 ```

### Create Page Index and About

 - Create the `pages` directory at the root of your project.

 - Create the `index.vue` file under the pages directory and copy below.

 ```ts
    <template>
        <div id="home-page">
            <div id="home-header">
                <h2>Home Page</h2>
            </div>
            <div id="home-container">
                <nav>
                <van-grid :border="false" :column-num="1">
                    <van-grid-item>
                        <NuxtLink to="/about" class="n-link-base">
                            About
                        </NuxtLink>
                    </van-grid-item>
                </van-grid>
                </nav>
            </div>
        </div>
    </template>
    <script setup lang="ts">
    definePageMeta({
        layout:"home",
        alias:["/home", "/"]
    })
    </script>
    <style scoped>
    #home-page {
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        display: flex;
        position: absolute; 
        z-index: 1;  
    }
    #home-header {
        position:absolute;
        width: 100%;
        top: 40px;
        text-align:center; 
    }
    #home-container {
        position: absolute;
        top: 96px;
        left: 0;
        right: 0;
        bottom: 0;
    }

    </style>
 ```

 - Create the `about.vue` file under the pages directory and copy below.

 ```ts
    <template>
        <div id="about-page">
            <header>
                <PageHeader :title="pageTitle"></PageHeader>
            </header>
            <div id="about-container">
                <p class="about-content-p">This is an example of how to use Capacitor Video Player plugin</p>
            </div>
        </div>
    </template>
    <script lang="ts">
    export default defineNuxtComponent({
        name: "about",
        alias: "/about",
        setup() {
            const pageTitle = "About Page";
            return {pageTitle}
        }
    });
    </script>
    <style scoped>
    #about-page {
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        display: flex;
        position: absolute;
        z-index: 1;   
    }
    #about-container {
        position: absolute;
        top: 96px;
        left: 0;
        right: 0;
        bottom: 0;
    }
    .about-content-p {
        padding: 2%;
    }
    </style>
 ```

 As you can see in the about page, a custom `PageHeader` component is used.

### Create Custom PageHeader Component 

 - Create the `components` directory at the root of your project.

 - Create the `PageHeader.vue` file which contains our custom page header that will be used throughout the application.

 ```ts
    <template>
        <div class="page-header">
            <NuxtLink to="/" class="page-header-link">
                Back
            </NuxtLink>
        <h2>{{ title }}</h2>
        </div>
    </template>
    <script lang="ts">
    export default defineNuxtComponent({
        name: 'PageHeader',
        props: ['title'],
        setup(props) {
            const title = props.title;
            return {title}
        }
    })
    </script>
    <style scoped>
    .page-header {
        position:absolute;
        width: 100%;
        top: 40px;
        display:flex;
        flex-direction: row;
        z-index: 1;
    }
    .page-header-link {
        padding-left: 1%;
        padding-right: 1%;
        margin-top: 28px;
    }
    .page-header h2 {
        padding-left: 4%;
        padding-right: 1%;
    }
    </style>
 ```

 - Now you can run the app and you must be able to navigate from the Home page to the About page and back to the Home page. Run the following command.

 ```bash
    npm run dev
 ```
### Install capacitor-video-player plugin

 - Execute the following command.

 ```bash
 npm i --save capacitor-video-player
 ```

### Create Video Utilities

 - Create the `interfaces` directory at the root of your project.

 - Create the `interfaces.ts` file under the interfaces directory which will contain the video model.

```ts
    export interface SubtitleOptions {
        backgroundColor?: string,
        fontSize?: number,
        foregroundColor?: string
    }
    export interface VideoModel {
        device: string,
        title?: string,
        description?: string,
        url: string,
        type: string,
        gif?: string,
        published_at: string,
        subtitle_url?: string,
        subtitle_langage?: string,
        subtitle_options?: SubtitleOptions
    }
```

 - Create the `assets` directory at the root of your project and the json subdirectory.

 - Create the `videolist.json` file under the json directory and fill in as follows.

 ```json
    {
        "videoList" : [
            {
                "title": "MIB2",
                "description": "Lorem ipsum dolor sit amet. Facilis recusandae ex eaque recusandae non voluptas veniam nam enim modi.",
                "url": "https://brenopolanski.github.io/html5-video-webvtt-example/MIB2.mp4",
                "subtitle_url": "https://brenopolanski.github.io/html5-video-webvtt-example/MIB2-subtitles-pt-BR.vtt",
                "subtitle_langage": "es",
                "subtitle_options":{"backgroundColor":"rgba(0,0,0,0)","fontSize":18,"foregroundColor":"rgba(255,0,0,1)"},
                "type": "mp4",
                "published_at": "July 4 2020 12:30",
                "device": "all"
            },
            {
                "title": "Big Buck Bunny",
                "description": "Eos rerum quia ex repellendus asperiores sit sunt aperiam sed amet adipisci nam dignissimos consectetur qui voluptatem voluptas.",
                "url": "https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8",
                "type": "m3u8",
                "published_at": "December 3 2021 17:50",
                "device": "all"
            }
        ]
    }
 ```

 - Create the `services` directory at the root of your project.

 - Create the `video.service.ts` file under the services directory and edit it as follows.

 ```ts
    import videoList from '@/assets/json/videolist.json';
    import type { VideoModel } from '@/interfaces/interfaces';
    import type { Ref } from 'vue';

    export default class VideoService {
        videoList: VideoModel[] = videoList.videoList;
        public async get(contentId: Ref<string | string[]>): Promise<any>{
            const index = Number(contentId.value);
            if(index <= this.videoList.length) {
                return Promise.resolve({content: this.videoList[index]});        
            } else {
                return Promise.resolve({});
            }
        }
    }
 ```

### Create Composables

 - Create the `composables` directory at the root of your project.

 - Create the `mystate.ts` file under the composables directory which will contain a state hook.

 ```ts
    // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
    export function useMyState(initialState: any): any {
    const state = ref(initialState);
    const setState = (newState: any) => {
        state.value = newState;
    };
    
    return [readonly(state), setState];
    }
 ```

 - Create the `videoplayer.ts` file under the composables directory which will contain the video player hook using the capacitor-video-player.

 ```ts
    import { CapacitorVideoPlayer } from 'capacitor-video-player';
    import { Capacitor } from '@capacitor/core';

    export interface VideoPlayerOuput  {
        result?: boolean;
        method?: string;
        value?: any;
        message?: string;
    }

    interface PlayerData {
        mode: string,
        url: string,
        playerId: string,
        componentTag: string, 
        subtitle?: string | null,
        language?: string | null,
        subtitleOptions?: any | null,
        rate?: number | null,
        exitOnEnd?: boolean | null,
        loopOnEnd?: boolean | null,
        pipEnabled?: boolean | null,
        bkmodeEnabled?: boolean | null,
        showControls?: boolean | null,
        displayMode?: string | null,
        width?: number | null,
        height?: number | null
    }

    export interface VideoPlayerHook {
        echo: (value: string) => Promise<{value: string}>;
        initPlayer: (mode: string, url: string, playerId: string,
            componenTag: string, subTitleUrl?: string, 
            subTitleLangage?: string, subTitleOptions?: any, 
            rate?: number, exitOnEnd?: boolean, loopOnEnd?: boolean,
            pipEnabled?: boolean, bkmodeEnabled?: boolean,
            showControls?: boolean, displayMode?: string,
            width?:number, height?:number) => 
            Promise<VideoPlayerOuput>;
        isPlaying: (playerId: string) => Promise<VideoPlayerOuput>;
        pause: (playerId: string) => Promise<VideoPlayerOuput>;
        play: (playerId: string) => Promise<VideoPlayerOuput>;
        getDuration: (playerId: string) => Promise<VideoPlayerOuput>;
        setVolume: (playerId: string, volume: number) => 
            Promise<VideoPlayerOuput>;
        getVolume: (playerId: string) => Promise<VideoPlayerOuput>;
        setMuted: (playerId: string, muted: boolean) => 
            Promise<VideoPlayerOuput>;
        getMuted: (playerId: string) => Promise<VideoPlayerOuput>;
        setCurrentTime: (playerId: string, seektime: number) => 
            Promise<VideoPlayerOuput>;
        getCurrentTime: (playerId: string) => Promise<VideoPlayerOuput>;
        stopAllPlayers: () => Promise<VideoPlayerOuput>;
        removeListeners: () => Promise<void>;
        getPlatform: () => Promise<{platform: string}>;
    }

    export function useVideoPlayer(emit: (event: "onPlay" | "onPause" | "onEnded" | "onExit" | "onReady", ...args: any[]) => void): VideoPlayerHook {
        const platform = Capacitor.getPlatform();
        const vpPlugin: any = CapacitorVideoPlayer;
        let playListener: any;
        let pauseListener: any;
        let endedListener: any;
        let exitListener: any;
        let readyListener: any;
        
        /**
         * Add Listeners
         */
        const addListeners = async (): Promise<void> => {
            playListener = await vpPlugin.addListener('jeepCapVideoPlayerPlay',
                (e: any) => {
                    const fromPlayerId = e.fromPlayerId;
                    const currentTime = e.currentTime
                    emit('onPlay', {fromPlayerId, currentTime})
                }, false);
            pauseListener = await vpPlugin.addListener('jeepCapVideoPlayerPause',
                (e: any) => {
                    const fromPlayerId = e.fromPlayerId;
                    const currentTime = e.currentTime
                    emit('onPause', {fromPlayerId, currentTime})
                }, false);
            endedListener = await vpPlugin.addListener('jeepCapVideoPlayerEnded',
                (e: any) => {
                    const fromPlayerId = e.fromPlayerId;
                    const currentTime = e.currentTime
                    emit('onEnded', {fromPlayerId, currentTime})
                }, false);
            exitListener = await vpPlugin.addListener('jeepCapVideoPlayerExit',
                (e: any) => {
                    const dismiss = e.dismiss;
                    emit('onExit', {dismiss})
                },false);
            readyListener = await vpPlugin.addListener('jeepCapVideoPlayerReady',
                (e: any) => {
                    const fromPlayerId = e.fromPlayerId;
                    const currentTime = e.currentTime
                    emit('onReady', {fromPlayerId, currentTime})
                }, false);
        }
        /**
         * Remove Listeners
         */
        const removeListeners = async (): Promise<void> => {
            await playListener.remove();
            await pauseListener.remove();
            await endedListener.remove();
            await exitListener.remove();
            await readyListener.remove();
        };
        /**
         * Echo value
         * @param value 
         */
        const echo = async (value: string): Promise<{value: string}> => {
            const ret = {value: ""};
            if(value) {
                const r = await vpPlugin.echo(value);
                if(r) {
                    return r;
                }
                return ret;    
            } else {
                ret.value = "Echo: failed";
                return ret;
            }
        };
        /**
         * Get Platform
         */
        const getPlatform = async (): Promise<any> => {
            return {platform: platform};
        };
        /**
         * Method initPlayer
         * Init the player
         * @param mode 
         * @param url 
         * @param playerId 
         * @param componentTag 
         * @param subTitleUrl 
         * @param subTitleLanguage 
         * @param subTitleOptions 
         * @param width 
         * @param height 
         * @returns 
         */
        const initPlayer = async (mode: string, url : string,
            playerId: string, componentTag: string,
            subTitleUrl?: string , subTitleLanguage?: string, subTitleOptions?: any,
            rate?: number, exitOnEnd?: boolean, loopOnEnd?: boolean,
            pipEnabled?: boolean, bkmodeEnabled?: boolean,
            showControls?: boolean, displayMode?: string) => {
            const playerData: PlayerData = {mode: mode, url: url, 
                playerId: playerId, componentTag: componentTag}
            playerData.subtitle = subTitleUrl != null ? subTitleUrl : null; 
            playerData.language = subTitleLanguage != null ? subTitleLanguage : null;
            if(subTitleOptions != null) {
                playerData.subtitleOptions = {};
                if(subTitleOptions.backgroundColor != null)
                    playerData.subtitleOptions.backgroundColor = subTitleOptions.backgroundColor;
                if(subTitleOptions.fontSize != null)
                    playerData.subtitleOptions.fontSize = subTitleOptions.fontSize;
                if(subTitleOptions.foregroundColor != null)
                    playerData.subtitleOptions.foregroundColor = subTitleOptions.foregroundColor;
                
            } else {
                playerData.subtitleOptions = null;
            }
            playerData.rate = rate != null ? rate : 1; 
            playerData.exitOnEnd = exitOnEnd != null ? exitOnEnd : true; 
            playerData.loopOnEnd = loopOnEnd != null ? loopOnEnd : false; 
            playerData.pipEnabled = pipEnabled != null ? pipEnabled : true; 
            playerData.bkmodeEnabled = bkmodeEnabled != null ? bkmodeEnabled : true; 
            playerData.displayMode = displayMode != null ? displayMode : 'portrait'; 

            const r = await vpPlugin.initPlayer(playerData);
            if (r) {
                if( typeof r.result != 'undefined') {
                    return r;
                }
            }
            return {result: false, method: "initPlayer", 
                    message: "initPlayer failed"};
        }; 
        /**
         * Method isPlaying 
         * @param playerId string
         */
        const isPlaying = async (playerId: string) => {
            const r = await vpPlugin.isPlaying({ playerId:playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "isPlaying",
                message: "isPlaying failed" };

        };

        /**
         * Method pause 
         * pause the videoplayer 
         * @param playerId string
         */
        const pause = async (playerId: string) => {
            const r = await vpPlugin.pause({ playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "pause",
                message: "pause failed" };

        };

        /**
         * Method play 
         * play the videoplayer 
         * @param playerId string
         */
        const play = async (playerId: string) => {
            const r = await vpPlugin.play({ playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "play",
                message: "play failed" };

        };

        /**
         * Method getDuration 
         * get the video duration
         * @param playerId string
         */
        const getDuration = async (playerId: string) => {
            const r = await vpPlugin.getDuration({ 
                playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "getDuration",
                message: "getDuration failed" };

        };

        /**
         * Method setVolume 
         * set the video volume
         * @param playerId string
         * @param volume number
         */
        const setVolume = async (playerId: string,
                volume: number) => {
            const r = await vpPlugin.setVolume({ playerId: playerId,
                volume: volume });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "setVolume",
                message: "setVolume failed" };

        };

        /**
         * Method getVolume 
         * get the video volume
         * @param playerId string
         */
        const getVolume = async (playerId: string) => {
            const r = await vpPlugin.getVolume({ 
                    playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
        return { result: false, method: "getVolume",
            message: "getVolume failed" };

        };

        /**
         * Method setMuted 
         * set the video muted parameter
         * @param playerId string
         * @param muted boolean
         */
        const setMuted = async (playerId: string,
                muted: boolean) => {
            const r = await vpPlugin.setMuted({ 
                    playerId: playerId, muted: muted });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "setMuted",
                message: "setMuted failed" };

        };

        /**
         * Method getMuted 
         * get the video muted parameter
         * @param playerId string
         */
        const getMuted = async (playerId: string) => {
            const r = await vpPlugin.getMuted({ playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "getMuted",
                message: "getMuted failed" };

        };

        /**
         * Method setCurrentTime 
         * set the video current time
         * @param playerId string
         * @param seektime number
         */
        const setCurrentTime = async (playerId: string,
                    seektime: number) => {
            const r = await vpPlugin.setCurrentTime({ 
                playerId: playerId, seektime: seektime });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "setCurrentTime",
                message: "setCurrentTime failed" };

        };

        /**
         * Method getCurrentTime 
         * get the video current time
         * @param playerId string
         */
        const getCurrentTime = async (playerId: string) => {
            const r = await vpPlugin.getCurrentTime({ 
                playerId: playerId });
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "getCurrentTime",
                message: "getCurrentTime failed" };

        };

        /**
         * Method stopAllPlayers
         * stop all players
         */
        const stopAllPlayers = async () => {
            const r = await vpPlugin.stopAllPlayers();
            if (r) {
                if (typeof r.result != 'undefined') {
                    return r;
                }
            }
            return { result: false, method: "stopAllPlayers",
                message: "stopAllPlayers failed" };

        };

        // Add Listeners
        addListeners();
        
        return { echo, initPlayer, isPlaying, play, pause, getDuration,
            setVolume, getVolume, setMuted, getMuted, setCurrentTime, 
            getCurrentTime, stopAllPlayers, removeListeners, getPlatform };
    }
 ```

### Create FullscreenVideoPlayer Component

This component will play the video by using the video player hook.
In the `components` directory, create the `FullscreenVideoPlayer.vue` file and copy the following.

```ts
    <template>
    <div id="fullscreen">
    </div>
    </template>
    <script lang="ts">
    export default defineNuxtComponent({
        name: 'FullscreenVideoPlayer',
        props: ['videoData'],
        emits: ['onPlay','onPause','onEnded', 'onExit','onReady'],
        setup(props, { emit }) {
        let ret: any = {};
        let params: any[] = [];
        const vpHook: VideoPlayerHook = useVideoPlayer(emit);
        const playVideo = async () => {
            params = [];
    
            if( props.videoData.subtitle_url != null && 
                props.videoData.subtitle_langage != null) {
            params = [...params,props.videoData.subtitle_url,props.videoData.subtitle_langage];
            }
            if(props.videoData.subtitle_options != null) {
            params = [...params,props.videoData.subtitle_options];
            }
            const args: any[] = [...params];
            ret = await vpHook.initPlayer("fullscreen",props.videoData.url,"fullscreen","div",...params);
            console.log(`ret : ${JSON.stringify(ret)}`)

        } 
        onMounted(async () => {
            await playVideo();
        });
        onUnmounted(async () => {
            await vpHook.removeListeners();
            await vpHook.stopAllPlayers();
        });
        }
    });
    </script>
 ```

### Create Viewer Video Page

This page will use the `FullscreenVideoPlayer` vue component based on the index of the video in the videoList.

 - Create the `viewervideo` directory under the `pages` directory.

 - Create the `[id].vue` page under the `viewervideo` directory, and copy the following code.

 ```ts
    <template>
    <div id="viewervideo-page">
        <header>
            <PageHeader :title="pageTitle"></PageHeader>
        </header>
        <div id="viewervideo-container">
            <van-grid :border="false" :column-num="1">
                <van-grid-item class="viewervideo-publish">
                    <p class="viewervideo-publish-date">
                    {{ prettyPublishedAt }}
                    </p>
                    <h2 class="viewervideo-publish-title">{{ videoContent.title }}</h2>
                </van-grid-item>
                <van-grid-item>
                    <span class="viewervideo-publish-description">{{ videoContent.description }}</span>
                </van-grid-item>
                <van-grid-item>
                    <van-button type="primary" block @click="playVideo()">
                        Watch
                    </van-button>
                </van-grid-item>
            </van-grid>
            <div v-if="isClicked">
                <FullscreenVideoPlayer @onPlay="handleOnPlay" @onPause="handleOnPause" @onEnded="handleOnEnded" @onExit="handleOnExit" @onReady="handleOnReady" :videoData="videoContent"></FullscreenVideoPlayer>
            </div>

        </div> 
    </div> 
    </template>
    <script lang="ts">
    import VideoService from '@/services/video.service';
    import type { VideoModel } from '~/interfaces/interfaces';
    export default defineNuxtComponent({
    name: "ViewerVideoPage",
    alias: "/viewervideo",
    setup() {
        // get the props id
        const route = useRoute();
        // build-up the page  title
        const videoRef = ref(route.params.id);
        const videoText = computed(() => {
            if(Number(videoRef.value) == 0) {
                return "MP4 Video"
            }
            if(Number(videoRef.value) == 1) {
                return "HLS Video"
            }
        });
        const pageTitle = `${videoText.value}`
        const videoService = new VideoService();
        const [isClicked, setIsClicked] = useMyState(false);
        const playerLeave = async () => {
            console.log (`>>> in [id].vue playerLeave`)
            setIsClicked(false);
        }
        const handleOnPlay = (e:any) => {
            console.log(`<<<< onPlay in ViewerVideo ${e.fromPlayerId} ct: ${e.currentTime}`);
        }
        const handleOnPause = (e:any) => {
            console.log(`<<<< onPause in ViewerVideo ${e.fromPlayerId} ct: ${e.currentTime}`)
        }
        const handleOnEnded = (e:any) => {
            console.log(`<<<< onEnded in ViewerVideo ${e.fromPlayerId} ct: ${e.currentTime}`)
            playerLeave();
        }
        const handleOnExit = (e:any) => {
            console.log(`<<<< onExit in ViewerVideo ${e.dismiss}`)
            playerLeave();
        }
        const handleOnReady = (e:any) => {
            console.log(`<<<< onReady in ViewerVideo ${e.fromPlayerId} ct: ${e.currentTime}`)
        }
        
        const videoContent = ref({} as VideoModel);

        const getVideoContent = async () => {
            const response = await videoService.get(videoRef);
            if ("content" in response) {
                videoContent.value = response.content;
            } else {
                console.log("Failed to load content");
                throw new Error("Failed to load content")
            }    
        }
        onMounted(async () => {
            await getVideoContent();
        });

        return {pageTitle, isClicked, setIsClicked, videoContent, handleOnPlay,
                handleOnPause, handleOnEnded, handleOnExit, handleOnReady};
    },
    computed: {
        prettyPublishedAt() {
        const strDate: string = this.videoContent.published_at;
        const d = new Date(strDate);
        return d.toLocaleString(undefined, {
            year: "numeric",
            month: "long",
            day: "numeric",
        });
        
        },
    },
    methods: {
        async playVideo() {
            this.setIsClicked(true);
        },
    }
    });
    </script>
    <style scoped>
    #viewervideo-page {
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    display: flex;
    position: absolute; 
    z-index: 1;  
    }
    #viewervideo-container {
    position: absolute;
    top: 96px;
    left: 0;
    right: 0;
    bottom: 0;
    }
    .viewervideo-publish {
    display:flex;
    flex-direction: row;
    }
    .viewervideo-publish-date {
    margin-top:3px;
    margin-bottom:3px;
    }
    .viewervideo-publish-title {
    margin-top:3px;
    margin-bottom:3px;
    }
    .viewervideo-publish-description {
    margin-top:3px;
    margin-bottom:3px;
    }

    </style> 
 ```

### Reference the VideoViewer Page in Index Page

 - open the `index.vue` file  and add the following lines afterthe `van-grid-item` containing the link to the `about` page.

 ```html
    <van-grid-item>
        <NuxtLink to="/viewervideo/0" class="n-link-base">
            MP4 Video
        </NuxtLink>
    </van-grid-item>
    <van-grid-item>
        <NuxtLink to="/viewervideo/1" class="n-link-base">
            HLS Video
        </NuxtLink>
    </van-grid-item>
 ```

### Run the Application in the Web Browser

 - Execute the following command.

 ```bash
 npm run dev
 ```
 - Open http://localhost:3000/ in your favorite browser.

 - Screenshot of the `Home` Page screen:

  <div align="center"><br><img src="/images/Vant-Nuxt-VideoPlayer-Home.png" width="50%" /></div><br>

 - Click on the `MP4 Video` link to go to the MP4 Video page.

 - Screenshot of the `MP4 Video` screen:

  <div align="center"><br><img src="/images/Vant-Nuxt-VideoPlayer-MP4.png" width="50%" /></div><br>

 - Click on `Watch` button and the video will start playing fullscreen.

Congratulations, the Web part of the application is now working

### Run the Application in iOS Device

 - Execute the following commands.

 ```bash
 npm i --save @capacitor/ios
 npm run generate
 npx cap add ios
 npx cap copy
 npx cap open ios
 ```

### Run the Application in Android Device

 - Execute the following commands.

 ```bash
 npm i --save @capacitor/android
 npm run generate
 npx cap add android
 npx cap copy
 npx cap open android
 ```

 - Add these lines in the `AndroidManifest.xml` file

 ```xml
    <meta-data
        android:name="com.google.android.gms.cast.framework.OPTIONS_PROVIDER_CLASS_NAME"
        android:value="com.google.android.exoplayer2.ext.cast.DefaultCastOptionsProvider">
    </meta-data>
 ```
 
 and 

 ```xml
     <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32"/>
 ```

 - Add this dependency in the `build.gradle(Project: android)` file under dependencies

 ```xml
    classpath 'com.google.gms:google-services:4.4.0'
 ```

 - Add this implementation in the `build.gradle(Module: app)`file under dependencies

 ```xml
     implementation 'com.google.android.gms:play-services-cast-framework:21.3.0'
 ```

### Conclusion

We have completed the Vant Nuxt Capacitor Video Player Example Tutorial using Nuxt, Vant and the capacitor-video-player plugin.

We learned how to implement the `capacitor-video-player` plugin in the Nuxt Framework using Vant, a lightweight, customizable Vue UI library, and Ionic Capacitor which add native functionality.

Enjoy your development from there.
