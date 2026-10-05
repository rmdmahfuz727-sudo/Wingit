name: Build APK

on:
  push:
    branches: [main, master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - name: Create the Android project
        run: |
          cat > settings.gradle <<'WINGIT_EOF'
          pluginManagement {
              repositories {
                  google()
                  mavenCentral()
                  gradlePluginPortal()
              }
          }

          dependencyResolutionManagement {
              repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
              repositories {
                  google()
                  mavenCentral()
              }
          }

          rootProject.name = "Wingit"
          include ':app'
          WINGIT_EOF
          cat > build.gradle <<'WINGIT_EOF'
          plugins {
              id 'com.android.application' version '8.7.3' apply false
          }
          WINGIT_EOF
          cat > gradle.properties <<'WINGIT_EOF'
          org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
          android.nonTransitiveRClass=true
          WINGIT_EOF
          mkdir -p app
          cat > app/build.gradle <<'WINGIT_EOF'
          plugins {
              id 'com.android.application'
          }

          android {
              namespace 'com.wingit.game'
              compileSdk 35

              defaultConfig {
                  applicationId 'com.wingit.game'
                  minSdk 24
                  targetSdk 35
                  versionCode 1
                  versionName '1.0'
              }

              buildTypes {
                  release {
                      minifyEnabled false
                  }
              }

              compileOptions {
                  sourceCompatibility JavaVersion.VERSION_17
                  targetCompatibility JavaVersion.VERSION_17
              }
          }
          WINGIT_EOF
          mkdir -p app/src/main
          cat > app/src/main/AndroidManifest.xml <<'WINGIT_EOF'
          <?xml version="1.0" encoding="utf-8"?>
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">

              <uses-permission android:name="android.permission.INTERNET" />

              <application
                  android:allowBackup="true"
                  android:icon="@drawable/ic_launcher"
                  android:label="Wingit"
                  android:theme="@style/AppTheme">

                  <activity
                      android:name=".MainActivity"
                      android:configChanges="orientation|screenSize|keyboardHidden|smallestScreenSize|screenLayout"
                      android:exported="true"
                      android:screenOrientation="portrait">
                      <intent-filter>
                          <action android:name="android.intent.action.MAIN" />
                          <category android:name="android.intent.category.LAUNCHER" />
                      </intent-filter>
                  </activity>
              </application>
          </manifest>
          WINGIT_EOF
          mkdir -p app/src/main/java/com/wingit/game
          cat > app/src/main/java/com/wingit/game/MainActivity.java <<'WINGIT_EOF'
          package com.wingit.game;

          import android.annotation.SuppressLint;
          import android.app.Activity;
          import android.graphics.Color;
          import android.os.Build;
          import android.os.Bundle;
          import android.view.View;
          import android.view.WindowInsets;
          import android.view.WindowInsetsController;
          import android.view.WindowManager;
          import android.webkit.WebSettings;
          import android.webkit.WebView;
          import android.webkit.WebViewClient;

          public class MainActivity extends Activity {
              private WebView web;

              @SuppressLint("SetJavaScriptEnabled")
              @Override
              protected void onCreate(Bundle savedInstanceState) {
                  super.onCreate(savedInstanceState);
                  getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

                  web = new WebView(this);
                  web.setBackgroundColor(Color.parseColor("#1D1740"));
                  web.setOverScrollMode(View.OVER_SCROLL_NEVER);
                  web.setVerticalScrollBarEnabled(false);
                  web.setHorizontalScrollBarEnabled(false);
                  web.setLongClickable(false);
                  web.setHapticFeedbackEnabled(false);
                  web.setOnLongClickListener(new View.OnLongClickListener() {
                      @Override
                      public boolean onLongClick(View v) {
                          return true;
                      }
                  });

                  WebSettings s = web.getSettings();
                  s.setJavaScriptEnabled(true);
                  s.setDomStorageEnabled(true);
                  s.setMediaPlaybackRequiresUserGesture(false);

                  web.setWebViewClient(new WebViewClient());
                  setContentView(web);
                  web.loadUrl("file:///android_asset/index.html");
              }

              @Override
              public void onWindowFocusChanged(boolean hasFocus) {
                  super.onWindowFocusChanged(hasFocus);
                  if (hasFocus) {
                      hideSystemBars();
                  }
              }

              @SuppressWarnings("deprecation")
              private void hideSystemBars() {
                  if (Build.VERSION.SDK_INT >= 30) {
                      getWindow().setDecorFitsSystemWindows(false);
                      WindowInsetsController c = getWindow().getInsetsController();
                      if (c != null) {
                          c.hide(WindowInsets.Type.systemBars());
                          c.setSystemBarsBehavior(WindowInsetsController.BEHAVIOR_SHOW_TRANSIENT_BARS_BY_SWIPE);
                      }
                  } else {
                      getWindow().getDecorView().setSystemUiVisibility(
                              View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                                      | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                                      | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                                      | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                                      | View.SYSTEM_UI_FLAG_FULLSCREEN
                                      | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);
                  }
              }

              @Override
              protected void onResume() {
                  super.onResume();
                  web.onResume();
              }

              @Override
              protected void onPause() {
                  web.onPause();
                  super.onPause();
              }

              @Override
              protected void onDestroy() {
                  if (web != null) {
                      web.destroy();
                  }
                  super.onDestroy();
              }
          }
          WINGIT_EOF
          mkdir -p app/src/main/res/values
          cat > app/src/main/res/values/styles.xml <<'WINGIT_EOF'
          <?xml version="1.0" encoding="utf-8"?>
          <resources>
              <color name="night">#1D1740</color>

              <style name="AppTheme" parent="android:Theme.Material.NoActionBar">
                  <item name="android:windowBackground">@color/night</item>
                  <item name="android:statusBarColor">@color/night</item>
                  <item name="android:navigationBarColor">@color/night</item>
              </style>
          </resources>
          WINGIT_EOF
          mkdir -p app/src/main/res/drawable
          cat > app/src/main/res/drawable/ic_launcher.xml <<'WINGIT_EOF'
          <?xml version="1.0" encoding="utf-8"?>
          <vector xmlns:android="http://schemas.android.com/apk/res/android"
              android:width="108dp"
              android:height="108dp"
              android:viewportWidth="108"
              android:viewportHeight="108">
              <path
                  android:fillColor="#1D1740"
                  android:pathData="M0,0h108v108h-108z" />
              <path
                  android:fillColor="#FF6B5E"
                  android:pathData="M28,54a26,22 0 1,0 52,0a26,22 0 1,0 -52,0" />
              <path
                  android:fillColor="#FFF1DC"
                  android:pathData="M40,62a14,9 0 1,0 28,0a14,9 0 1,0 -28,0" />
              <path
                  android:fillColor="#FFB347"
                  android:pathData="M36,56a10,6 0 1,0 20,0a10,6 0 1,0 -20,0" />
              <path
                  android:fillColor="#FFFFFF"
                  android:pathData="M58,47a6,6 0 1,0 12,0a6,6 0 1,0 -12,0" />
              <path
                  android:fillColor="#1D1740"
                  android:pathData="M62,47a2.5,2.5 0 1,0 5,0a2.5,2.5 0 1,0 -5,0" />
              <path
                  android:fillColor="#FFB347"
                  android:pathData="M78,50L92,55L78,60z" />
          </vector>
          WINGIT_EOF
          mkdir -p app/src/main/assets
          cat > app/src/main/assets/index.html <<'WINGIT_EOF'
          <!doctype html>
          <html lang="en">
          <head>
          <meta charset="utf-8">
          <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
          <title>Wingit</title>
          <link rel="preconnect" href="https://fonts.googleapis.com">
          <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
          <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Bagel+Fat+One&family=DM+Mono:wght@500&display=swap">
          <style>
          /* Layout: slim top bar, one tall dusk-lit playfield scaled to fit the screen, a hint line beneath.
             Single look by choice: a game that lives at dusk, so it keeps its own palette in any theme. */
          :root {
            --night: #1d1740;
            --dusk: #3b2a6b;
            --glow: #ff9a6b;
            --copper: #c8693a;
            --brass: #f1c05a;
            --cream: #fff1dc;
            --coral: #ff6b5e;
            --font-display: "Bagel Fat One", "Arial Rounded MT Bold", Impact, sans-serif;
            --font-mono: "DM Mono", ui-monospace, Menlo, Consolas, monospace;
            color-scheme: dark;
          }
          html, body { height: 100%; }
          body {
            box-sizing: border-box;
            margin: 0;
            padding-inline: 16px;
            padding-block: 12px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            background: var(--night);
            color: var(--cream);
            font-family: var(--font-mono);
            overflow: hidden;
          }
          .bar { display: flex; align-items: center; justify-content: space-between; gap: 12px; }
          .mark { margin: 0; font: 400 28px/1 var(--font-display); color: var(--coral); letter-spacing: 0.02em; }
          .snd {
            font: 500 12px/1 var(--font-mono);
            color: var(--cream);
            background: transparent;
            border: 1px solid rgba(255, 241, 220, 0.35);
            border-radius: 999px;
            padding: 11px 16px;
            cursor: pointer;
          }
          .snd:hover { border-color: var(--brass); }
          .snd:focus-visible { outline: 2px solid var(--brass); outline-offset: 2px; }
          .stage {
            flex: 1;
            min-height: 0;
            min-width: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            touch-action: none;
            user-select: none;
            -webkit-user-select: none;
            -webkit-tap-highlight-color: transparent;
          }
          canvas {
            display: block;
            max-width: 100%;
            border-radius: 14px;
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.45);
            cursor: pointer;
          }
          .hint { margin: 0; text-align: center; font-size: 12px; line-height: 1.4; color: rgba(255, 241, 220, 0.72); }
          .sr { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }
          </style>
          </head>
          <body>

          <header class="bar">
            <h1 class="mark">Wingit</h1>
            <button id="snd" class="snd" type="button" aria-pressed="true">Sound on</button>
          </header>
          <main id="stage" class="stage">
            <canvas id="game" width="360" height="560" role="img" aria-label="Wingit playfield. Tap, click or press space to flap."></canvas>
          </main>
          <p class="hint">Space, ↑ or tap to flap. Fly through the gaps in the copper pipes.</p>
          <div id="live" class="sr" aria-live="polite"></div>

          <script>
          (() => {
            const W = 360, H = 560, GROUND = 76, FLOOR = H - GROUND;
            const GRAV = 0.36, FLAP = -6.4, MAXFALL = 8.5, SPEED = 2.3;
            const GAP = 150, PW = 58, CAPH = 22, SPACING = 200;
            const BX = 96, BR = 12, HR = 10;

            const canvas = document.getElementById('game');
            const ctx = canvas.getContext('2d');
            const stage = document.getElementById('stage');
            const sndBtn = document.getElementById('snd');
            const live = document.getElementById('live');
            const reduced = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;

            const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
            const mod = (a, m) => ((a % m) + m) % m;

            // ---------- state ----------
            let view = 1;
            let state = 'ready';
            let bird, pipes, score, scroll, frames = 0, shake = 0, flapAnim = 0;
            let deadAt = 0, newBest = false, onGround = false;
            let best = 0;
            try { best = parseInt(localStorage.getItem('wingit-best') || '0', 10) || 0; } catch (e) {}

            // stars (deterministic)
            const stars = [];
            let seed = 7;
            const rnd = () => (seed = (seed * 16807) % 2147483647) / 2147483647;
            for (let i = 0; i < 46; i++) stars.push({ x: rnd() * W, y: rnd() * (FLOOR * 0.5), r: 0.6 + rnd() * 1.1, p: rnd() * 6 });
            const clouds = [{ x: 30, y: 110, s: 1 }, { x: 190, y: 190, s: 0.7 }, { x: 320, y: 70, s: 0.85 }, { x: 460, y: 150, s: 1.1 }];

            function reset() {
              bird = { y: H * 0.42, vy: 0, rot: 0, t: 0 };
              pipes = [];
              score = 0;
              scroll = scroll || 0;
              shake = 0;
              flapAnim = 0;
              onGround = false;
              newBest = false;
            }

            function spawnPipe(x) {
              const prev = pipes.length ? pipes[pipes.length - 1].gapY : FLOOR / 2;
              const lo = Math.max(GAP / 2 + 60, prev - 150);
              const hi = Math.min(FLOOR - 60 - GAP / 2, prev + 150);
              pipes.push({ x, gapY: lo + Math.random() * (hi - lo), passed: false });
            }

            function start() {
              state = 'playing';
              pipes = [];
              spawnPipe(W + 80);
              live.textContent = 'Go';
            }

            function flap() {
              bird.vy = FLAP;
              flapAnim = 12;
              sfx.flap();
            }

            function die() {
              if (state === 'dead') return;
              state = 'dead';
              deadAt = performance.now();
              bird.vy = Math.min(bird.vy, -2.5);
              if (!reduced) shake = 9;
              if (score > best) {
                best = score;
                newBest = true;
                try { localStorage.setItem('wingit-best', String(best)); } catch (e) {}
              }
              sfx.hit();
              live.textContent = 'Game over. Score ' + score + '. Best ' + best + '.';
            }

            function action() {
              audioInit();
              if (state === 'ready') { start(); flap(); }
              else if (state === 'playing') flap();
              else if (performance.now() - deadAt > 650) { reset(); start(); flap(); }
            }

            // ---------- sound ----------
            let ac = null, muted = false;
            function audioInit() {
              if (muted) return;
              if (!ac) {
                try { ac = new (window.AudioContext || window.webkitAudioContext)(); } catch (e) { ac = null; }
              }
              if (ac && ac.state === 'suspended') ac.resume();
            }
            function tone(f1, f2, dur, type, vol, delay) {
              if (!ac || muted) return;
              const t = ac.currentTime + (delay || 0);
              const o = ac.createOscillator(), g = ac.createGain();
              o.type = type;
              o.frequency.setValueAtTime(f1, t);
              o.frequency.exponentialRampToValueAtTime(f2, t + dur);
              g.gain.setValueAtTime(vol, t);
              g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
              o.connect(g);
              g.connect(ac.destination);
              o.start(t);
              o.stop(t + dur + 0.02);
            }
            const sfx = {
              flap() { tone(380, 640, 0.09, 'triangle', 0.09); },
              point() { tone(660, 660, 0.06, 'square', 0.04); tone(990, 990, 0.1, 'square', 0.04, 0.06); },
              hit() { tone(240, 50, 0.28, 'sawtooth', 0.09); }
            };
            sndBtn.addEventListener('click', () => {
              muted = !muted;
              sndBtn.textContent = muted ? 'Sound off' : 'Sound on';
              sndBtn.setAttribute('aria-pressed', String(!muted));
              if (!muted) audioInit();
              sndBtn.blur();
            });

            // ---------- input ----------
            stage.addEventListener('pointerdown', (e) => {
              if (e.target === sndBtn) return;
              e.preventDefault();
              action();
            });
            window.addEventListener('keydown', (e) => {
              if (e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW') {
                if (e.target && e.target.tagName === 'BUTTON') return;
                e.preventDefault();
                if (!e.repeat) action();
              }
            });

            // ---------- sizing ----------
            function fit() {
              const aw = stage.clientWidth, ah = stage.clientHeight;
              const s = Math.max(0.2, Math.min(aw / W, ah / H));
              const cw = Math.floor(W * s), ch = Math.floor(H * s);
              const dpr = Math.min(window.devicePixelRatio || 1, 3);
              canvas.style.width = cw + 'px';
              canvas.style.height = ch + 'px';
              canvas.width = Math.round(cw * dpr);
              canvas.height = Math.round(ch * dpr);
              view = canvas.width / W;
            }
            if (window.ResizeObserver) new ResizeObserver(fit).observe(stage);
            window.addEventListener('resize', fit);

            // ---------- simulation ----------
            function hitRect(rx, ry, rw, rh) {
              const cx = clamp(BX, rx, rx + rw), cy = clamp(bird.y, ry, ry + rh);
              const dx = BX - cx, dy = bird.y - cy;
              return dx * dx + dy * dy < HR * HR;
            }

            function step() {
              frames++;
              if (flapAnim > 0) flapAnim--;
              shake *= 0.84;
              if (state !== 'dead') scroll += SPEED;
              bird.t++;

              if (state === 'ready') {
                bird.y = H * 0.42 + Math.sin(bird.t * 0.08) * 8;
                bird.rot = 0;
                return;
              }

              if (state === 'dead') {
                if (!onGround) {
                  bird.vy = Math.min(bird.vy + GRAV, MAXFALL);
                  bird.y += bird.vy;
                  bird.rot = Math.min(bird.rot + 0.12, 1.5);
                  if (bird.y >= FLOOR - BR) { bird.y = FLOOR - BR; onGround = true; }
                }
                return;
              }

              bird.vy = Math.min(bird.vy + GRAV, MAXFALL);
              bird.y += bird.vy;
              bird.rot = clamp(bird.vy * 0.07, -0.45, 1.2);
              if (bird.y - BR < 0) { bird.y = BR; bird.vy = 0; }

              for (const p of pipes) p.x -= SPEED;
              if (pipes.length && pipes[0].x < -PW - 12) pipes.shift();
              const last = pipes[pipes.length - 1];
              if (!last || last.x < W + PW - SPACING) spawnPipe(W + PW);

              for (const p of pipes) {
                if (!p.passed && p.x + PW < BX) {
                  p.passed = true;
                  score++;
                  sfx.point();
                  live.textContent = String(score);
                }
                if (BX + HR > p.x - 4 && BX - HR < p.x + PW + 4) {
                  const gt = p.gapY - GAP / 2, gb = p.gapY + GAP / 2;
                  if (hitRect(p.x, -60, PW, gt - CAPH + 60) || hitRect(p.x - 4, gt - CAPH, PW + 8, CAPH) ||
                      hitRect(p.x - 4, gb, PW + 8, CAPH) || hitRect(p.x, gb + CAPH, PW, FLOOR - gb - CAPH)) {
                    die();
                    return;
                  }
                }
              }
              if (bird.y + BR >= FLOOR) {
                bird.y = FLOOR - BR;
                onGround = true;
                die();
              }
            }

            // ---------- drawing ----------
            function rr(x, y, w, h, r) {
              ctx.beginPath();
              if (ctx.roundRect) ctx.roundRect(x, y, w, h, r); else ctx.rect(x, y, w, h);
            }

            function label(str, x, y, size, fill, o) {
              o = o || {};
              ctx.font = o.mono ? '500 ' + size + 'px "DM Mono", ui-monospace, monospace'
                                : size + 'px "Bagel Fat One", "Arial Rounded MT Bold", Impact, sans-serif';
              ctx.textAlign = o.align || 'center';
              ctx.textBaseline = 'alphabetic';
              ctx.lineJoin = 'round';
              if (o.stroke) { ctx.lineWidth = o.sw || 6; ctx.strokeStyle = o.stroke; ctx.strokeText(str, x, y); }
              ctx.fillStyle = fill;
              ctx.fillText(str, x, y);
            }

            function hills(off, base, amp, f1, f2, color) {
              ctx.fillStyle = color;
              ctx.beginPath();
              ctx.moveTo(0, FLOOR);
              for (let x = 0; x <= W; x += 6) {
                const u = x + off;
                ctx.lineTo(x, base - amp * (Math.sin(u * f1) + 0.55 * Math.sin(u * f2 + 1.3)));
              }
              ctx.lineTo(W, FLOOR);
              ctx.closePath();
              ctx.fill();
            }

            function star(cx, cy, r, ri) {
              ctx.beginPath();
              for (let i = 0; i < 10; i++) {
                const a = -Math.PI / 2 + i * Math.PI / 5, rad = i % 2 ? ri : r;
                const x = cx + Math.cos(a) * rad, y = cy + Math.sin(a) * rad;
                if (i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
              }
              ctx.closePath();
            }

            function drawBackdrop() {
              const sky = ctx.createLinearGradient(0, 0, 0, FLOOR);
              sky.addColorStop(0, '#1d1740');
              sky.addColorStop(0.5, '#43296f');
              sky.addColorStop(0.8, '#b8517a');
              sky.addColorStop(1, '#ff9a6b');
              ctx.fillStyle = sky;
              ctx.fillRect(0, 0, W, H);

              for (const s of stars) {
                ctx.globalAlpha = 0.35 + 0.35 * Math.sin(frames * 0.03 + s.p);
                ctx.fillStyle = '#fff1dc';
                ctx.beginPath();
                ctx.arc(s.x, s.y, s.r, 0, 6.2832);
                ctx.fill();
              }
              ctx.globalAlpha = 1;

              const glow = ctx.createRadialGradient(250, FLOOR - 62, 8, 250, FLOOR - 62, 140);
              glow.addColorStop(0, 'rgba(255,176,116,0.6)');
              glow.addColorStop(1, 'rgba(255,176,116,0)');
              ctx.fillStyle = glow;
              ctx.fillRect(0, 0, W, FLOOR);
              ctx.fillStyle = '#ffd39a';
              ctx.beginPath();
              ctx.arc(250, FLOOR - 62, 38, 0, 6.2832);
              ctx.fill();

              for (const c of clouds) {
                const cx = mod(c.x - scroll * 0.12, W + 220) - 110;
                ctx.fillStyle = 'rgba(255,190,170,0.2)';
                ctx.beginPath();
                ctx.ellipse(cx, c.y, 34 * c.s, 11 * c.s, 0, 0, 6.2832);
                ctx.ellipse(cx + 22 * c.s, c.y - 8 * c.s, 24 * c.s, 12 * c.s, 0, 0, 6.2832);
                ctx.ellipse(cx - 22 * c.s, c.y - 4 * c.s, 20 * c.s, 9 * c.s, 0, 0, 6.2832);
                ctx.fill();
              }

              hills(scroll * 0.15, FLOOR - 40, 22, 0.011, 0.027, '#4a2d73');
              hills(scroll * 0.35 + 90, FLOOR - 14, 16, 0.019, 0.041, '#2d2058');
            }

            function drawPipe(p) {
              const gt = p.gapY - GAP / 2, gb = p.gapY + GAP / 2;
              const body = ctx.createLinearGradient(p.x, 0, p.x + PW, 0);
              body.addColorStop(0, '#8a3b20');
              body.addColorStop(0.28, '#e58e56');
              body.addColorStop(0.6, '#c8693a');
              body.addColorStop(1, '#78321a');
              const cap = ctx.createLinearGradient(p.x - 4, 0, p.x + PW + 4, 0);
              cap.addColorStop(0, '#9a6f1f');
              cap.addColorStop(0.3, '#f8d98a');
              cap.addColorStop(0.65, '#dca640');
              cap.addColorStop(1, '#8c6419');
              ctx.strokeStyle = 'rgba(24,10,34,0.6)';
              ctx.lineWidth = 2;

              ctx.fillStyle = body;
              ctx.fillRect(p.x, -10, PW, gt - CAPH + 10);
              ctx.strokeRect(p.x, -10, PW, gt - CAPH + 10);
              ctx.fillRect(p.x, gb + CAPH, PW, FLOOR - gb - CAPH);
              ctx.strokeRect(p.x, gb + CAPH, PW, FLOOR - gb - CAPH);
              ctx.fillStyle = 'rgba(255,255,255,0.16)';
              ctx.fillRect(p.x + 9, -10, 4, gt - CAPH + 10);
              ctx.fillRect(p.x + 9, gb + CAPH, 4, FLOOR - gb - CAPH);

              ctx.fillStyle = cap;
              rr(p.x - 4, gt - CAPH, PW + 8, CAPH, 5); ctx.fill(); ctx.stroke();
              rr(p.x - 4, gb, PW + 8, CAPH, 5); ctx.fill(); ctx.stroke();
              ctx.fillStyle = 'rgba(80,45,10,0.55)';
              for (const rx of [p.x + 3, p.x + PW - 3]) {
                ctx.beginPath(); ctx.arc(rx, gt - CAPH / 2, 1.8, 0, 6.2832); ctx.fill();
                ctx.beginPath(); ctx.arc(rx, gb + CAPH / 2, 1.8, 0, 6.2832); ctx.fill();
              }
            }

            function drawGround() {
              ctx.fillStyle = '#2a1a3d';
              ctx.fillRect(0, FLOOR, W, GROUND);
              ctx.fillStyle = '#e9a35a';
              ctx.fillRect(0, FLOOR, W, 8);
              ctx.fillStyle = '#c47f3c';
              ctx.fillRect(0, FLOOR + 8, W, 3);
              ctx.fillStyle = '#35224d';
              const off = mod(scroll, 40);
              for (let x = -40 - off; x < W + 40; x += 40) {
                ctx.beginPath();
                ctx.moveTo(x, FLOOR + 11);
                ctx.lineTo(x + 20, FLOOR + 11);
                ctx.lineTo(x + 4, H);
                ctx.lineTo(x - 16, H);
                ctx.closePath();
                ctx.fill();
              }
            }

            function drawBird() {
              ctx.save();
              ctx.translate(BX, bird.y);
              ctx.rotate(bird.rot);
              ctx.lineWidth = 2;
              ctx.strokeStyle = 'rgba(24,10,34,0.65)';

              ctx.fillStyle = '#d94a42';
              ctx.beginPath();
              ctx.moveTo(-12, -1); ctx.lineTo(-24, -7); ctx.lineTo(-22, 3); ctx.lineTo(-24, 9); ctx.lineTo(-11, 5);
              ctx.closePath(); ctx.fill(); ctx.stroke();

              ctx.fillStyle = '#d94a42';
              ctx.beginPath();
              ctx.moveTo(-3, -11); ctx.lineTo(-1, -19); ctx.lineTo(3, -12); ctx.lineTo(7, -18); ctx.lineTo(8, -10);
              ctx.closePath(); ctx.fill(); ctx.stroke();

              ctx.fillStyle = '#ff6b5e';
              ctx.beginPath(); ctx.ellipse(0, 0, 16, 13, 0, 0, 6.2832); ctx.fill(); ctx.stroke();
              ctx.fillStyle = '#fff1dc';
              ctx.beginPath(); ctx.ellipse(3, 5, 10, 6.5, 0.1, 0, 6.2832); ctx.fill();

              let wing;
              if (state === 'ready') wing = Math.sin(bird.t * 0.22) * 0.7;
              else if (flapAnim > 0) wing = -0.9 + 1.4 * (1 - flapAnim / 12);
              else wing = 0.35;
              ctx.save();
              ctx.translate(-4, 1);
              ctx.rotate(wing);
              ctx.fillStyle = '#ffb347';
              ctx.beginPath(); ctx.ellipse(-4, 0, 10, 5.5, 0, 0, 6.2832); ctx.fill(); ctx.stroke();
              ctx.restore();

              ctx.fillStyle = '#ffb347';
              ctx.beginPath(); ctx.moveTo(13, -3); ctx.lineTo(23, 1); ctx.lineTo(13, 5); ctx.closePath(); ctx.fill(); ctx.stroke();

              if (state === 'dead') {
                ctx.strokeStyle = '#1d1740'; ctx.lineWidth = 2; ctx.lineCap = 'round';
                ctx.beginPath(); ctx.moveTo(4, -8); ctx.lineTo(11, -1); ctx.moveTo(11, -8); ctx.lineTo(4, -1); ctx.stroke();
              } else {
                ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(7, -4, 4.4, 0, 6.2832); ctx.fill(); ctx.stroke();
                ctx.fillStyle = '#1d1740'; ctx.beginPath(); ctx.arc(8.4, -4, 2.1, 0, 6.2832); ctx.fill();
              }
              ctx.restore();
            }

            function drawReady(now) {
              label('WINGIT', W / 2, 124, 60, '#ff6b5e', { stroke: '#1d1740', sw: 10 });
              label("Tap to flap. Don't touch the pipes.", W / 2, 158, 13, '#fff1dc', { mono: true });
              const pulse = 0.65 + 0.35 * Math.sin(now * 0.005);
              ctx.globalAlpha = pulse;
              label('TAP TO START', W / 2, 345, 22, '#f1c05a', { stroke: '#1d1740', sw: 6 });
              ctx.globalAlpha = 1;
              if (best > 0) label('Best ' + best, W / 2, 375, 13, '#fff1dc', { mono: true });
            }

            function drawDead(now) {
              const since = now - deadAt;
              const f = Math.max(0, 1 - since / 180);
              if (f > 0) { ctx.fillStyle = 'rgba(255,255,255,' + (f * 0.7) + ')'; ctx.fillRect(0, 0, W, H); }
              const a = clamp((since - 350) / 250, 0, 1);
              if (a <= 0) return;
              ctx.save();
              ctx.globalAlpha = a;
              ctx.translate(0, (1 - a) * 18);
              rr(60, 160, 240, 230, 16);
              ctx.fillStyle = 'rgba(29,23,64,0.94)'; ctx.fill();
              ctx.lineWidth = 3; ctx.strokeStyle = '#f1c05a'; ctx.stroke();
              label('GAME OVER', W / 2, 206, 30, '#ff6b5e');

              const medals = [[40, '#8fe3e0'], [30, '#ffcf4a'], [20, '#cfd6e6'], [10, '#cd7f32']];
              let col = null;
              for (const m of medals) if (score >= m[0]) { col = m[1]; break; }
              ctx.beginPath(); ctx.arc(112, 292, 30, 0, 6.2832);
              if (col) {
                ctx.fillStyle = col; ctx.fill();
                ctx.lineWidth = 3; ctx.strokeStyle = 'rgba(24,10,34,0.55)'; ctx.stroke();
                star(112, 292, 17, 7); ctx.fillStyle = 'rgba(255,255,255,0.75)'; ctx.fill();
              } else {
                ctx.setLineDash([4, 5]); ctx.lineWidth = 2; ctx.strokeStyle = 'rgba(255,241,220,0.4)'; ctx.stroke(); ctx.setLineDash([]);
                label('10', 112, 298, 18, 'rgba(255,241,220,0.4)');
              }

              label('SCORE', 164, 258, 11, 'rgba(255,241,220,0.7)', { mono: true, align: 'left' });
              label(String(score), 164, 294, 36, '#fff1dc', { align: 'left' });
              label('BEST', 164, 322, 11, 'rgba(255,241,220,0.7)', { mono: true, align: 'left' });
              label(String(best), 164, 352, 28, '#f1c05a', { align: 'left' });
              if (newBest) label('NEW BEST', 112, 346, 11, '#f1c05a', { mono: true });
              ctx.restore();

              if (since > 650) {
                ctx.globalAlpha = 0.65 + 0.35 * Math.sin(now * 0.006);
                label('TAP TO RETRY', W / 2, 432, 22, '#f1c05a', { stroke: '#1d1740', sw: 6 });
                ctx.globalAlpha = 1;
              }
            }

            function draw(now) {
              ctx.setTransform(view, 0, 0, view, 0, 0);
              ctx.save();
              if (shake > 0.3) ctx.translate((Math.random() - 0.5) * shake, (Math.random() - 0.5) * shake);
              drawBackdrop();
              for (const p of pipes) drawPipe(p);
              drawGround();
              drawBird();
              ctx.restore();
              ctx.setTransform(view, 0, 0, view, 0, 0);
              if (state === 'playing' || state === 'dead') {
                label(String(score), W / 2, 74, 56, '#fff1dc', { stroke: '#1d1740', sw: 8 });
              }
              if (state === 'ready') drawReady(now);
              if (state === 'dead') drawDead(now);
            }

            // ---------- loop ----------
            let last = 0, acc = 0;
            const DT = 1000 / 60;
            function frame(t) {
              if (!last) last = t;
              acc += Math.min(t - last, 100);
              last = t;
              while (acc >= DT) { step(); acc -= DT; }
              draw(performance.now());
              requestAnimationFrame(frame);
            }

            scroll = 0;
            reset();
            fit();
            requestAnimationFrame(frame);
          })();
          </script>

          </body>
          </html>
          WINGIT_EOF

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Accept Android SDK licenses
        run: yes | "${ANDROID_HOME:-$ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager" --licenses > /dev/null || true

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.10.2'

      - name: Build debug APK
        run: gradle --no-daemon assembleDebug

      - name: Name the APK
        run: cp app/build/outputs/apk/debug/app-debug.apk Wingit.apk

      - uses: actions/upload-artifact@v4
        with:
          name: Wingit-apk
          path: Wingit.apk
