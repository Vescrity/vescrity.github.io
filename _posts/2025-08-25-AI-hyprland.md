---
layout: post
title: gemini 说 hyprland 长这样
categories: [other]
---

<div style="background-color: rgb(30, 30, 46); width: 100%; height: 80vh; position: relative; overflow: hidden; font-family: &quot;DejaVu Sans Mono&quot;, monospace; color: rgb(205, 214, 244); box-shadow: rgba(0, 0, 0, 0.4) 0px 0px 30px inset; display: flex; justify-content: center; align-items: center; --darkreader-inline-bgcolor: var(--darkreader-background-1e1e2e, #000006); --darkreader-inline-color: var(--darkreader-text-cdd6f4, #dfffff); --darkreader-inline-boxshadow: var(--darkreader-background-00000066, rgba(0, 0, 0, 0.4)) 0px 0px 30px inset;" data-darkreader-inline-bgcolor="" data-darkreader-inline-color="" data-darkreader-inline-boxshadow="">
    <style>
        .hyprland-screen-inner {
            width: 100%;
            height: 100%;
            position: relative;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .hyprland-window {
            background-color: #313244;
            border: 1px solid #585b70;
            border-radius: 8px;
            box-shadow: 0 5px 20px rgba(0,0,0,0.5);
            position: absolute;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            border-top-left-radius: 8px;
            border-top-right-radius: 8px;
        }
        .window-header {
            background-color: #24273a;
            height: 30px;
            border-bottom: 1px solid #585b70;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 0 10px;
            border-top-left-radius: 7px;
            border-top-right-radius: 7px;
            flex-shrink: 0;
        }
        .window-title {
            color: #cdd6f4;
            font-size: 0.9em;
            font-weight: 500;
        }
        .window-controls {
            display: flex;
            gap: 6px;
        }
        .close-btn, .min-btn, .max-btn {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            display: inline-block;
        }
        .close-btn { background-color: #f38ba8; } /* Red */
        .min-btn { background-color: #fab387; } /* Orange */
        .max-btn { background-color: #a6e3a1; } /* Green */

        .window-content {
            padding: 12px;
            height: calc(100% - 30px);
            overflow: auto;
            font-size: 0.9em;
            color: #9399b2;
            display: flex;
            flex-direction: column;
            justify-content: flex-start; /* Content starts from top */
            flex-grow: 1;
        }

        .window-content p {
            margin: 0 0 8px 0;
            white-space: pre-wrap;
            word-break: break-all;
        }
        .window-content p:last-child { margin-bottom: 0; }
        .cursor { animation: blinker 1s step-end infinite; }
        @keyframes blinker { 50% { opacity: 0; } }

        /* Specific color accents for different terminals */
        .hyprland-window:nth-child(2) .window-header { background-color: #3b4252; }
        .hyprland-window:nth-child(2) .window-title { color: #88c0d0; } /* Cyan */
        .hyprland-window:nth-child(2) .window-content { color: #81a1c1; }

        .hyprland-window:nth-child(3) .window-header { background-color: #434c5e; }
        .hyprland-window:nth-child(3) .window-title { color: #ebcb8b; } /* Yellow */
        .hyprland-window:nth-child(3) .window-content { color: #d8dee9; }

        .hyprland-window:nth-child(4) .window-header { background-color: #343d4f; }
        .hyprland-window:nth-child(4) .window-title { color: #bf616a; } /* Red */
        .hyprland-window:nth-child(4) .window-content { color: #b48ead; } /* Purple */

        .hyprland-window:nth-child(5) .window-header { background-color: #2e3440; }
        .hyprland-window:nth-child(5) .window-title { color: #9d82d8; } /* Purple */
        .hyprland-window:nth-child(5) .window-content { color: #8fbcbb; } /* Teal */
    </style>
    <div class="hyprland-screen-inner">
        <!-- Terminal 1 -->
        <div class="hyprland-window" style="top: 80px; left: 200px; width: 550px; height: 350px; z-index: 5;">
            <div class="window-header">
                <span class="window-title">Terminal [1] - Zsh</span>
                <div class="window-controls">
                    <span class="close-btn"></span>
                    <span class="min-btn"></span>
                    <span class="max-btn"></span>
                </div>
            </div>
            <div class="window-content">
                <p>user@hyprland:~ $ <span class="cursor">_</span></p>
                <p>user@hyprland:~ $ <span style="color: rgb(166, 227, 161); --darkreader-inline-color: var(--darkreader-text-a6e3a1, #bcffb4);" data-darkreader-inline-color="">ls -l</span></p>
                <p>total 12</p>
                <p>drwxr-xr-x 2 user user 4096 Mar 15 10:30 Documents</p>
                <p>drwxr-xr-x 3 user user 4096 Mar 15 10:31 Downloads</p>
                <p>-rw-r--r-- 1 user user  567 Mar 15 11:15 hypr_config.sh</p>
                <p style="margin-top: auto;">user@hyprland:~ $ <span class="cursor">_</span></p>
            </div>
        </div>

        <!-- Terminal 2 -->
        <div class="hyprland-window" style="top: 40px; left: 100px; width: 450px; height: 280px; z-index: 4;">
            <div class="window-header">
                <span class="window-title">Browser - Arch Wiki</span>
                <div class="window-controls">
                    <span class="close-btn"></span>
                    <span class="min-btn"></span>
                    <span class="max-btn"></span>
                </div>
            </div>
            <div class="window-content">
                <p>Navigating: <span style="color: rgb(250, 179, 135); --darkreader-inline-color: var(--darkreader-text-fab387, #ffd489);" data-darkreader-inline-color="">wiki.archlinux.org</span></p>
                <p>...</p>
                <p><span style="color: rgb(147, 153, 178); --darkreader-inline-color: var(--darkreader-text-9399b2, #d9cebd);" data-darkreader-inline-color="">Loading "Hyprland" article...</span></p>
                <p>...</p>
                <p style="margin-top: auto;"><span class="cursor">_</span></p>
            </div>
        </div>

        <!-- Terminal 3 -->
        <div class="hyprland-window" style="top: 60px; left: 550px; width: 450px; height: 280px; z-index: 4;">
            <div class="window-header">
                <span class="window-title">Editor - script.sh</span>
                <div class="window-controls">
                    <span class="close-btn"></span>
                    <span class="min-btn"></span>
                    <span class="max-btn"></span>
                </div>
            </div>
            <div class="window-content">
                <p><span style="color: rgb(250, 179, 135); --darkreader-inline-color: var(--darkreader-text-fab387, #ffd489);" data-darkreader-inline-color="">#!/bin/bash</span></p>
                <p>echo "Hello, Hyprland!"</p>
                <p># Update system</p>
                <p><span style="color: rgb(166, 227, 161); --darkreader-inline-color: var(--darkreader-text-a6e3a1, #bcffb4);" data-darkreader-inline-color="">sudo pacman -Syu --noconfirm</span></p>
                <p>echo "System updated."</p>
                <p style="margin-top: auto;"><span class="cursor">_</span></p>
            </div>
        </div>

        <!-- Terminal 4 -->
        <div class="hyprland-window" style="top: 380px; left: 150px; width: 450px; height: 280px; z-index: 4;">
            <div class="window-header">
                <span class="window-title">System Monitor (btop)</span>
                <div class="window-controls">
                    <span class="close-btn"></span>
                    <span class="min-btn"></span>
                    <span class="max-btn"></span>
                </div>
            </div>
            <div class="window-content">
                <p><span style="color: rgb(166, 227, 161); --darkreader-inline-color: var(--darkreader-text-a6e3a1, #bcffb4);" data-darkreader-inline-color="">CPU:</span> 18% | <span style="color: rgb(250, 179, 135); --darkreader-inline-color: var(--darkreader-text-fab387, #ffd489);" data-darkreader-inline-color="">RAM:</span> 2.8GB / 16GB</p>
                <p><span style="color: rgb(136, 192, 208); --darkreader-inline-color: var(--darkreader-text-88c0d0, #9df5ff);" data-darkreader-inline-color="">Load:</span> 0.55 0.68 0.72</p>
                <p><span style="color: rgb(230, 233, 239); --darkreader-inline-color: var(--darkreader-text-e6e9ef, #ffffff);" data-darkreader-inline-color="">Processes:</span> 135 | <span style="color: rgb(191, 97, 106); --darkreader-inline-color: var(--darkreader-text-bf616a, #f97380);" data-darkreader-inline-color="">Threads:</span> 512</p>
                <p>--------------------------------------------</p>
                <p>USER PID %CPU %MEM SWAP COMMAND</p>
                <p>user 12345 5.2 1.5 0.0 firefox</p>
                <p>user 54321 2.1 0.8 0.0 kitty</p>
                <p style="margin-top: auto;"><span class="cursor">_</span></p>
            </div>
        </div>

        <!-- Terminal 5 -->
        <div class="hyprland-window" style="top: 400px; left: 590px; width: 450px; height: 280px; z-index: 4;">
            <div class="window-header">
                <span class="window-title">Music - cmus</span>
                <div class="window-controls">
                    <span class="close-btn"></span>
                    <span class="min-btn"></span>
                    <span class="max-btn"></span>
                </div>
            </div>
            <div class="window-content">
                <p>Now Playing:</p>
                <p><span style="color: rgb(243, 139, 168); font-weight: bold; --darkreader-inline-color: var(--darkreader-text-f38ba8, #ff92c2);" data-darkreader-inline-color="">"Sunset Drive"</span></p>
                <p>Artist: <span style="color: rgb(216, 222, 233); --darkreader-inline-color: var(--darkreader-text-d8dee9, #f9ffff);" data-darkreader-inline-color="">Synthwave Masters</span></p>
                <p>Album: <span style="color: rgb(216, 222, 233); --darkreader-inline-color: var(--darkreader-text-d8dee9, #f9ffff);" data-darkreader-inline-color="">Neon Nights</span></p>
                <p>Vol: [======----] 60%</p>
                <p style="margin-top: auto;">[ &lt;&lt; ] [ ▶ Play ] [ &gt;&gt; ] [ || Pause ]</p>
                <p><span class="cursor">_</span></p>
            </div>
        </div>
    </div>
</div>

