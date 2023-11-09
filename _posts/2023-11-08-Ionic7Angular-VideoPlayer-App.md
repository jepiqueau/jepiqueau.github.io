# Ionic 7 VideoPlayer App Example Tutorial using Angular and capacitor-video-player plugin
---

*last updated on November 9, 2023 by Quéau Jean Pierre*

In this tutorial, we will learn how to create a simple Ionic7/Angular video player application by implementing the capacitor-video-player plugin to display a list of videos with some data and play a selected video in fullscreen mode.

By using Ionic Capacitor 5, the application will use a single source code for the creation of web browser, PWA and native (iOS, Android) applications.

The application can be found at [ionic7-angular-videoplayer-app](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/Videoplayer/ionic7-angular-videoplayer-app)



## Table of Contents

---
 - [Create New Ionic/Angular Application](#create-new-ionicangular-application)
 - [Create a Data Service](#create-a-data-service)
 - [Show the Video List on Home Page](#show-the-video-list-on-home-page)
 - [Create the videoitem component](#create-the-videoitem-component)
 - [Serve the Application](#serve-the-application)
 - [Install the Capacitor Video Player Plugin](#install-the-capacitor-video-player-plugin)
 - [Create the Viewvideo Page](#create-the-viewvideo-page)
 - [Capacitor Video Player API methods](#capacitor-video-player-api-methods)
 - [Run the Web Application](#run-the-web-application)
 - [Prepare for Native Applications](#prepare-for-native-applications)
 - [Run the iOS Application](#run-the-ios-application)
 - [Run the Android Application](#run-the-android-application)
 - [Conclusion](#conclusion)



### Create New Ionic/Angular Application

 - To create a new Ionic/Angular application, we will use the `Ionic CLI`. Install it with the following command.

 ```bash
    npm install -g @ionic/cli
 ```

 - After installation, create the new Ionic/Angular application with the following command.

 ```bash
    ionic start ionic7-angular-videoplayer-app list --type=angular --capacitor
 ```

    Select `Standalone Components` as build method when asked.

 - When the new application is successfully created it will show a message to serve an application using `ionic serve`.

 - Go to the newly created application.

 ```bash
    cd ./ionic7-angular-videoplayer-app
 ```

 - Open the application in your favorite code editor

### Create a Data Service

 - if a `data.service.ts`file exists on the `app/services`, please update it with the below code. Otherwise create a `services` folder under the `app`folder and create a `data.service.ts` file with your favorite code editor.

 - Then replace/create the code in `data.service.ts`.

 ```ts
 import { Injectable } from '@angular/core';

export interface Video {
  device: string;
  type: string;
  title: string;
  url: string;
  thumb?: string;
  note: string;
  subtitle?: string,
  stlang?: string,
}


@Injectable({
  providedIn: 'root'
})
export class DataService {
  private videos: Video[] = [
    {
      type: "mp4",
      device: "all",
      url: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4",
      note: "By Blender Foundation",
      thumb: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg",
      title: "Big Buck Bunny"
    },
    {
      type: "mp4",
      device: "all",
      url: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4",
      note: "By Blender Foundation",
      thumb: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/ElephantsDream.jpg",
      title: "Elephant Dream"
    },
    {
      type: "mp4",
      device: "all",
      url: "https://media.bernat.ch/videos/2019-self-hosted-videos-subtitles/progressive.mp4",
      note: "Blender Animation Studio",
      thumb: "https://d2pzklc15kok91.cloudfront.net/images/posters/2019-self-hosted-videos-subtitles.3b55d44c736235.jpg",
      title: "327",
      subtitle: "https://media.bernat.ch/videos/2019-self-hosted-videos-subtitles.en.vtt",
      stlang: "en"
    },
    {
      type: "aws",
      device: "all",
      url: "https://universo-dev-a-m.s3.amazonaws.com/779970/fe774806dbe7ad042c24ce522b7b46594f16c66e",
      note: "Custom",
      title: "AWS video test",
    },
    {
      type: "hls",
      device: "all",
      url: "https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8",
      note: "By Blender Foundation",
      title: "Big Buck Bunny",
    },
    {
      type: "mpd",
      device: "android",
      url: "https://bitmovin-a.akamaihd.net/content/MI201109210084_1/mpds/f08e80da-bf1d-4e3d-8899-f0f6155f6efa.mpd",
      note: "",
      title: "MPD video test",
    },
    {
      type: "ism",
      device: "android",
      url: "https://test.playready.microsoft.com/smoothstreaming/SSWSS720H264/SuperSpeedway_720.ism/manifest",
      note: "",
      title: "ISM video test",
    },
    {
      type: "webm",
      device: "android",
      url: "https://upload.wikimedia.org/wikipedia/commons/transcoded/f/f1/Sintel_movie_4K.webm/Sintel_movie_4K.webm.720p.webm",
      note: "Blender Foundation",
      title: "ISM video test",
    },
  ];

  constructor() { }

  public getVideos(platform: string): Video[] {
    if (platform === 'web') {
      // Filter videos for Web platform (device: 'all')
      return this.videos.filter((video) => video.device === 'all');
    } else if (platform === 'android') {
      // Filter videos for Android platform (device: 'all' or 'android')
      return this.videos.filter((video) => video.device === 'all' || video.device === 'android');
    } else if (platform === 'ios') {
      // Filter videos for iOS platform (device: 'all' or 'ios')
      return this.videos.filter((video) => video.device === 'all' || video.device === 'ios');
    } else {
      return [];
    }
  }
  public getVideo(index:number): Video | null {
    if(index <= this.videos.length) {
      return this.videos[index];
    } else {
      return null;
    }
  }

}
 ```

 - Suppress the `message` folder and the `view-message` folder if they exist. 

### Show the Video List on Home Page

Fetch the Video list from the `DataService` and render it on the Home page.

 - Open the `home.page.html` under the `home`folder and replace the code with this.

 ```html
    <ion-header [translucent]="true">
        <ion-toolbar>
            <ion-title>
            Video List
            </ion-title>
        </ion-toolbar>
    </ion-header>
    <ion-content [fullscreen]="true">
        <ion-refresher slot="fixed" (ionRefresh)="refresh($event)">
            <ion-refresher-content></ion-refresher-content>
        </ion-refresher>
        <ion-header collapse="condense">
            <ion-toolbar>
            <ion-title size="large">
                Video List
            </ion-title>
            </ion-toolbar>
        </ion-header>

        <ion-list>
            <app-videoitem *ngFor="let video of getVideos(); index as i" [video]="video" [index]="i"></app-videoitem>
        </ion-list>
    </ion-content>
 ``` 

  - Then open the `home.page.ts` under the `home`folder and replace the code with this.

  ```ts
    import { CommonModule } from '@angular/common';
    import { Component, inject, AfterViewInit } from '@angular/core';
    import { RefresherCustomEvent, IonHeader, IonToolbar, IonTitle, IonContent, IonRefresher, IonRefresherContent, IonList } from '@ionic/angular/standalone';
    import { VideoitemComponent } from '../components/videoitem/videoitem.component';
    import { DataService, Video} from '../services/data.service';
    import { Device } from '@capacitor/device';

    @Component({
        selector: 'app-home',
        templateUrl: 'home.page.html',
        styleUrls: ['home.page.scss'],
        standalone: true,
        imports: [CommonModule, IonHeader, IonToolbar, IonTitle, IonContent, IonRefresher, IonRefresherContent, IonList, VideoitemComponent],
    })
    export class HomePage implements AfterViewInit {
        public platform: string = 'web';
        private data = inject(DataService);


        constructor() {}

        async ngAfterViewInit() {
            const info = await Device.getInfo();
            this.platform = info.platform;
            console.log(`*** in home platform: ${this.platform}`);

        }


        refresh(ev: any) {
            setTimeout(() => {
            (ev as RefresherCustomEvent).detail.complete();
            }, 3000);
        }

        getVideos(): Video[] {
            return this.data.getVideos(this.platform);
        }
    }
  ```

as you saw in the code we need to define a `Videoitem` component to render a Video element.

### Create the Videoitem Component

 - For the creation of the `Videoitem` component we will use the `Ionic CLI`.

 ```bash
    ionic generate component components/videoitem 
 ```

 - When the `Videoitem`has been successfully generated, open the `videoitem.component.html` file in your favorite code editor and replace the code.

 ```html
    <ion-item>
    <ion-avatar slot="start">
        <img [src]="video!.thumb" />
    </ion-avatar>
    <div class="div-item">
        <ion-label>{{video!.title}}</ion-label>
        <ion-note>{{video!.note}}</ion-note>
    </div>
    <ion-button slot="end" fill="clean" size="large" (click)="play(index!)">
        <ion-icon name="play-circle-outline"></ion-icon>
    </ion-button>
    </ion-item>
 ```

 - Then open the `videoitem.component.scss` file and define the css for the `div-item` class.

 ```css
    .div-item  {
        display: flex;
        flex-direction: column;
    }
 ``` 

 - Now create the code related to this component by opening the `videoitem.component.ts` file.

 ```ts
    import { CommonModule } from '@angular/common';
    import { ChangeDetectionStrategy, Component, OnInit, Input } from '@angular/core';
    import { RouterLink } from '@angular/router';
    import { IonItem, IonLabel, IonIcon, IonAvatar, IonButton, IonNote } from '@ionic/angular/standalone';
    import { addIcons } from 'ionicons';
    import { playCircleOutline } from 'ionicons/icons';
    import { Video } from '../../services/data.service';
    import { Router } from '@angular/router';

    @Component({
    selector: 'app-videoitem',
    templateUrl: './videoitem.component.html',
    styleUrls: ['./videoitem.component.scss'],
    changeDetection: ChangeDetectionStrategy.OnPush,
    standalone: true,
    imports: [CommonModule, RouterLink, IonItem, IonLabel, IonIcon,
                IonAvatar, IonButton, IonNote],
    })
    export class VideoitemComponent implements OnInit {
        @Input() video?: Video;
        @Input() index?: number;
        platform: string = 'web';

        constructor(private route: Router) {
                addIcons({ playCircleOutline });
        }

        ngOnInit() {}


        async play(index: number) {
            this.route.navigate([`/viewvideo/${index}`]);
        }
    }

 ```

### Serve the Application

To get the platform, we use the Capacitor Device plugin, so we first need to install it.

 - Install @capacitor/device

 ```bash
    npm i --save @capacitor/device
 ```
 
 - Remove the route to the `message/:id` if it exists in the `app.route.ts` file.

 ```ts
-   {
-        path: 'message/:id',
-        loadComponent: () =>
-        import('./view-message/view-message.page').then((m) => m.ViewMessagePage),
-    },
 ```

 - Run the app

 ```bash
    npm run start
 ```

 - The application will look like has below in the Web browser.

  <div align="center"><br><img src="/images/Ionic7-Angular-VideoList.png" width="50%" /></div><br>


### Install the Capacitor Video Player Plugin

 - Install capacitor-video-player plugin using below command :

 ```bash
    npm install --save capacitor-video-player@latest
 ``` 

### Create the Viewvideo Page

 - Create the `viewvideo` page which implements the `capacitor-video-player` to show a selected video full screen. For this we are using the `Ionic CLI`.

 ```bash
 ionic generate page pages/viewvideo
 ```

 - If the page `viewvideo` was successfully created, open the `viewvideo.page.html` and copy the code into it.

 ```html
    <ion-header [translucent]="true">
        <ion-toolbar>
            <ion-buttons slot="start">
            <ion-back-button defaultHref="/home"></ion-back-button>
            </ion-buttons>
            <ion-title>viewvideo</ion-title>
        </ion-toolbar>
    </ion-header>

    <ion-content [fullscreen]="true">
        <ion-header collapse="condense">
            <ion-toolbar>
            <ion-title size="large">viewvideo</ion-title>
            </ion-toolbar>
        </ion-header>
        <!-- Mandatory id="fullscreen"  -->
        <div id="fullscreen" slot="fixed">
        </div>
    </ion-content>
 ```

    I draw your attention to the <div id="fullscreen"> tag, which is mandatory for the capacitor video player to work well.

 - Open the `viewvideo.page.scss` copy the following css.

 ```css 
    #fullscreen {
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        background-color: #000000;
    }
 ```

 - Then, open the `viewvideo.page.ts` and replace the code by the following.

 ```ts
    import { Component, OnInit, AfterViewInit, inject } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { FormsModule } from '@angular/forms';
    import { IonicModule } from '@ionic/angular';
    import { ActivatedRoute, Router } from '@angular/router';
    import { Device } from '@capacitor/device';
    import { CapacitorVideoPlayer } from 'capacitor-video-player';
    import { DataService, Video} from '../../services/data.service';

    @Component({
        selector: 'app-viewvideo',
        templateUrl: './viewvideo.page.html',
        styleUrls: ['./viewvideo.page.scss'],
        standalone: true,
        imports: [IonicModule, CommonModule, FormsModule]
    })
    export class ViewvideoPage implements OnInit, AfterViewInit {
        private data = inject(DataService);
        private videoIndex: number = 0;
        private videoPlayer: any;
        private platform: string = 'web';
        private video: Video | null = {} as Video;

        private handlerPlay: any;
        private handlerPause: any;
        private handlerEnded: any;
        private handlerReady: any;
        private handlerExit: any;

        constructor(private route: ActivatedRoute,
            private navRoute: Router) { }

        //***********************************
        // Component Lifecycle hook methods *
        //***********************************

        ngOnInit() {
            // Subscribe to the params observable to read the id parameter
            this.route.params.subscribe(params => {
            // Extract the 'id' parameter from the route
            this.videoIndex = +params['id'];
            console.log(this.videoIndex);
            });
        }

        async ngAfterViewInit() {
            // Get the selected video
            this.video = this.data.getVideo(this.videoIndex);

            // Define the platform and the video player
            const info = await Device.getInfo();
            this.platform = info.platform;
            this.videoPlayer = CapacitorVideoPlayer;

            // add plugin listeners
            await this.addListenersToPlayerPlugin();
            const props: any = {};

            // Initialize the video player
            if (this.video!.url != null) {
            props.mode = "fullscreen";
            props.url = this.video!.url;
            props.playerId = 'fullscreen';
            props.componentTag = 'app-viewvideo';
            if(this.video!.subtitle != null) props.subtitle = this.video!.subtitle;
            if(this.video!.stlang != null) props.stlang = this.video!.stlang;
            const res: any = await this.videoPlayer.initPlayer(props);
            }

        }

        async ngOnDestroy(): Promise<void> {
            // Remove the plugin listeners
            await this.handlerPlay.remove();
            await this.handlerPause.remove();
            await this.handlerEnded.remove();
            await this.handlerReady.remove();
            await this.handlerExit.remove();
            await this.videoPlayer.stopAllPlayers();
            return;
        }

        // *******************
        // Private Functions *
        // *******************

        // Define the plugin listeners
        private async addListenersToPlayerPlugin(): Promise<void> {
            this.handlerPlay = await this.videoPlayer.addListener('jeepCapVideoPlayerPlay',
            (data: any) => {
                const fromPlayerId = data.fromPlayerId;
                const currentTime = data.currentTime;
                console.log(`<<<< onPlay in ViewerVideo ${fromPlayerId} ct: ${currentTime}`);
            }, false);
            this.handlerPause = await this.videoPlayer.addListener('jeepCapVideoPlayerPause',
            (data: any) => {
                const fromPlayerId = data.fromPlayerId;
                const currentTime = data.currentTime;
                console.log(`<<<< onPause in ViewerVideo ${fromPlayerId} ct: ${currentTime}`);
            }, false);
            this.handlerEnded = await this.videoPlayer.addListener('jeepCapVideoPlayerEnded',
            (data: any) => {
                const fromPlayerId = data.fromPlayerId;
                const currentTime = data.currentTime;
                console.log(`<<<< onEnded in ViewerVideo ${fromPlayerId} ct: ${currentTime}`);
                this.playerLeave();
            }, false);
            this.handlerExit = await this.videoPlayer.addListener('jeepCapVideoPlayerExit',
            (data: any) => {
                const dismiss = data.dismiss ;
                console.log(`<<<< onExit in ViewerVideo ${dismiss}`);
                this.playerLeave();
            }, false);
            this.handlerReady = await this.videoPlayer.addListener('jeepCapVideoPlayerReady',
            (data: any) => {
                const fromPlayerId = data.fromPlayerId;
                const currentTime = data.currentTime;
                console.log(`<<<< onReady in ViewerVideo ${fromPlayerId} ct: ${currentTime}`);
            }, false);
            return;
        }

        // Action when the player ended or exit
        private playerLeave() {
            this.navRoute.navigate(['/home']);
            return;
        }
    }
 ```

 - Finally, open the `app.routes.ts` file and update the path.

 ```ts
    path: 'viewvideo'
 ```

 with 
 
 ```ts
    path: 'viewvideo/:id'
 ```

 In this `viewvideo` page, we saw how to

  - Initialize the `capcitor-video-player` with the `initPlayer` API method which has some options as arguments. We have used only few of them.

    - The complete list of options is given below.

    ```json
        {
            /**
            * Player mode
            *  - "fullscreen"
            *  - "embedded" (Web only)
            */
            mode?: string;
            /**
            * The url of the video to play
            */
            url?: string;
            /**
            * The url of subtitle associated with the video
            */
            subtitle?: string;
            /**
            * The language of subtitle
            * see https://github.com/libyal/libfwnt/wiki/Language-Code-identifiers
            */
            language?: string;
            /**
            * SubTitle Options
            */
            subtitleOptions?: SubTitleOptions;
            /**
            * Id of DIV Element parent of the player
            */
            playerId?: string;
            /**
            * Initial playing rate
            */
            rate?: number;
            /**
            * Exit on VideoEnd (iOS, Android)
            * default: true
            */
            exitOnEnd?: boolean;
            /**
            * Loop on VideoEnd when exitOnEnd false (iOS, Android)
            * default: false
            */
            loopOnEnd?: boolean;
            /**
            * Picture in Picture Enable (iOS, Android)
            * default: true
            */
            pipEnabled?: boolean;
            /**
            * Background Mode Enable (iOS, Android)
            * default: true
            */
            bkmodeEnabled?: boolean;
            /**
            * Show Controls Enable (iOS, Android)
            * default: true
            */
            showControls?: boolean;
            /**
            * Display Mode ["portrait", "landscape"] (iOS, Android)
            * default: "portrait"
            */
            displayMode?: string;
            /**
            * Component Tag or DOM Element Tag (React app)
            */
            componentTag?: string;
            /**
            * Player Width (mode "embedded" only)
            */
            width?: number;
            /**
            * Player height (mode "embedded" only)
            */
            height?: number;
            /**
            * Headers for the request (iOS, Android)
            * by Manuel García Marín (https://github.com/PhantomPainX)
            */
            headers?: {
                [key: string]: string;
            };
            /**
            * Title shown in the player (Android)
            * by Manuel García Marín (https://github.com/PhantomPainX)
            */
            title?: string;
            /**
            * Subtitle shown below the title in the player (Android)
            * by Manuel García Marín (https://github.com/PhantomPainX)
            */
            smallTitle?: string;
            /**
            * ExoPlayer Progress Bar and Spinner color (Android)
            * by Manuel García Marín (https://github.com/PhantomPainX)
            * Must be a valid hex color code
            * default: #FFFFFF
            */
            accentColor?: string;
            /**
            * Chromecast enable/disable (Android)
            * by Manuel García Marín (https://github.com/PhantomPainX)
            * default: true
            */
            chromecast?: boolean;
            /**
            * Artwork url to be shown in Chromecast player
            * by Manuel García Marín (https://github.com/PhantomPainX)
            * default: ""
            */
            artwork?: string;
        } 
    ```

 - Set the Capacitor Video Player plugin events.

    - jeepCapVideoPlayerPlay : Emitted when the video starts to play
    - jeepCapVideoPlayerPause: Emitted when the video is paused
    - jeepCapVideoPlayerReady: Emitted when the video is ready to play
    - jeepCapVideoPlayerEnded: Emitted when the video has ended
    - jeepCapVideoPlayerExit: Emitted when the exit player button is pressed

 - Use the `jeepCapVideoPlayerEnded` and `jeepCapVideoPlayerExit` events to leave the `viewvideo` page and root back to the `home` page.

### Capacitor Video Player API methods

 - Below find the list of the API methods.

    - initPlayer(options)       Initialize a video player
    - isPlaying(options)        Give the video player's status
    - play(options)             Start playing a video
    - pause(options)            Pause the video
    - getDuration()             Get the video's duration
    - getCurrentTime(options)   Get the video's current time
    - setCurrentTime(options)   Set the current time to seek
    - getVolume(options)        Get the video's volume
    - setVolume(options)        Set the video's volume
    - getMuted(options)         Get the video's muted
    - setMuted(options)         Set the video's muted
    - setRate()                 Set the video's rate
    - getRate(options)          Get the video's rate
    - stopAllPlayers()          Stop all players playing
    - showController()          Show controller
    - isControllerIsFullyVisible() Give the controller's status
    - exitPlayer()              Exit the player
 
### Run the Web Application

 - Run the app

 ```bash
    npm run start
 ```

 - The application will look like has below in the Web browser.

    <video width="90%"  controls>
      <source src="/videos/IonicAngularVideoPlayerAppWeb.mp4" type="video/mp4">
    </video><br>

### Prepare for Native Applications

 - Install @capacitor/ios and @capacitor/android.

 ```bash
    npm install @capacitor/ios @capacitor/android
 ```

 - Edit `appId` in the `capacitor.config.ts` with YOUR_APP_ID.

 - Edit `appName` in the `capacitor.config.ts` with YOUR_APP_NAME.

 - Create a production build.

 ```bash
    npm run build --prod
 ```
 
 - Add iOS platform.

 ```bash
    npx cap add ios
 ```

 - Add Android platform.

 ```bash
    npx cap add android
 ```
 
 - Copy the Ionic build to iOS and Android platforms.

 ```bash
    npx cap copy
 ```
 
### Run the iOS Application

 ```bash
    npx cap open ios
 ```

 - The application will look like has below in the iOS Device.

    <video width="50%"  controls>
      <source src="/videos/Ionic7AngularVideoPlayerAppIOS.mov" type="video/mp4">
    </video><br>


### Run the Android Application

 ```bash
    npx cap open android
 ```

 - In AndroidStudio got to Preferences->Build,Execution,Deployment->Build Tools->Gradle
   - modify Gradle JDK to `17 Oracle OpenJDK version 17.0.7`. 
   - click on `Apply`and `OK`.

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

 - In `variables.gradle` file modify
  
 ```xml
 coreSplashScreenVersion = '1.0.1'
 ```
 - Go to File->Sync Project with Gradle files.

 - Build and run the application

 - The application will look like has below in the AndroidDevice.

    <video width="50%"  controls>
      <source src="/videos/Ionic7AngularVideoPlayerAppAndroid.mov" type="video/mp4">
    </video><br>


### Conclusion

We have completed the Ionic 7 VideoPlayer App Example Tutorial using Angular framework and the capacitor-video-player plugin.

We learned how to implement the `capacitor-video-player` plugin in the Angular Framework using Ionic UI component toolkit, and Ionic Capacitor which add native functionality.

Enjoy your development from there.
