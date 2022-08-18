<h2>FoxQL - Simple Chat</h2>

<script>
    import foxql from "@foxql/foxql-peer";
    import { onMount } from 'svelte';
    import { messages } from '$lib/store.js';

    let inputValue = ''

    const node = new foxql({
        maxNodeCount: 50, // max connection limit 
        maxCandidateCallTime: 750, // how long to wait for a response from a candidate node
        powPoolingTime: 900,
        bridgeServer: {
            host: "https://foxql-bridge.herokuapp.com", // which bridge server to use
        },
        dappAlias: 'demo-chat-app'
    });

    node.setMetaData({
        name: "test-node",
        description: "test-desc",
    });

    node.on("message", async ({ nodeId, message,...webRTCSender }, simulate = false) => {
        if (simulate) {
            messages.set([...$messages, {
                sender: nodeId,
                message: 'CONNECTED!'
            }])
            messages.set([...$messages, {
                sender: nodeId,
                message: message
            }])
            console.log('This message getting websocket channel.')
            if(typeof message != 'string') return false;
            if(typeof message.length > 128) return false;

            return true; // accept webRTC connection.
        } 
        console.log('This message getting webrtc channel.')
        // webRTC message.
        messages.set([...$messages, {
            sender: webRTCSender.sender.nodeId,
            message: message
        }])
        
        // if u reply? use this.reply()
    });

    async function handleSend()
    {
        messages.set([...$messages, {
            sender: 'ME',
            message: inputValue
        }])
        
        const answer = await node.ask({
            transportPackage: {
                p2pChannelName: "message",
                message: inputValue,
            },
            livingTime: 1200,
            stickyNode: true,
            localWork: false
        });
        inputValue = ''
        console.log(answer)
    }

    onMount(()=> {
        node.start();
        console.log(node)
        window.p2p = node
    })
</script>

<ul>
    {#if $messages.length <= 0}
        <h3>Waiting new message :)</h3>
    {/if}
    {#each $messages as {sender, message}}
        <li><b>{sender}</b>: {message}</li>
    {/each}
</ul>

<input type = "text" placeholder = "your message" bind:value="{inputValue}">
<button on:click="{handleSend}">Send</button>
<style>
    ul {
        border: 1px dashed #333;
    }
    input {
        width: 40%;
    }
</style>