<script>
    import {GlobalCSS} from 'figma-plugin-ds-svelte';
    import {Button, Input, Label, SelectMenu, Textarea, Icon, IconPlay, IconResolveFilled, IconWarning, IconButton} from 'figma-plugin-ds-svelte';
    import {Checkbox} from 'figma-plugin-ds-svelte';
    import {OnboardingTip} from 'figma-plugin-ds-svelte';

    let serverAddress = "http://localhost:9020";
    let correctAddress = null

    let copyToClipboard = true;
    let resetDecollisionMap = true;
    let separateComposablesForComponents = true;

    let currentSelectedName = null;
    let rootSelectionCount = 0;
    let generatedOutput = null;

    $: disabled = currentSelectedName == null;
    $: connectionIcon = correctAddress == null ? IconPlay : (correctAddress === true ? IconResolveFilled : IconWarning)
    let errors = [];

    function generatePressed() {
        parent.postMessage({
            pluginMessage: {
                'type': 'generate',
                'serverAddress': serverAddress,
                'copyToClipboard': copyToClipboard,
                'separateComposablesForComponents': separateComposablesForComponents,
                'currentSelected': rootSelectionCount
            }
        }, '*');
    }

    function close() {
        parent.postMessage({pluginMessage: {'type': 'close'}}, '*')
    }

    function testConnection() {
        // parent.postMessage({pluginMessage: {'type': 'testConnection'}}, '*')
        sendMessageToJavaBackend(JSON.stringify({ test: true}))
    }

    function connectionError(message) {
        correctAddress = false
    }

    function sendMessageToJavaBackend(body) {
        let request = new XMLHttpRequest()
        // This link has random lorem ipsum text
        request.open('POST', serverAddress)
        request.responseType = 'text'
        request.onload = function () {
            correctAddress = true
            generatedOutput = request.response;
        };
        request.onerror = (e) => {
            connectionError(e);
        }
        request.onabort = (e) => {
            connectionError(e);
        }

        request.send(body)
    }

    onmessage = (event) => {
        if (event.data.pluginMessage.type === "newNodeSelection") {
            currentSelectedName = event.data.pluginMessage.selectionName;
            rootSelectionCount = event.data.pluginMessage.rootSelectionCount;
        }
        if (event.data.pluginMessage.type === "post") {
            //TODO: Avoid as many internal JSON serial and deserialzing
            sendMessageToJavaBackend(JSON.stringify({
                rootiestNode: JSON.parse(event.data.pluginMessage.data),
                copyToClipboard: copyToClipboard,
                resetDecollisionMap: resetDecollisionMap,
                separateComposablesForEachComponent: separateComposablesForComponents
            }));
        }
        if (event.data.pluginMessage.type === "error") {

        }
    }


</script>


<div class="pl-xxsmall pr-xxsmall">

    <h4 class="mt-xxsmall">Jetpack to Compose</h4>
    <p class="px">The <a href="https://github.com/CaelumF/FigmaToCompose">server</a> should be running</p>
    <hr class="mt-xsmall" style="border-bottom: none;">

    <label for="server">Server address</label>
    <div class="flex column ml-xxsmall">
        <Input id="server" bind:value={serverAddress} borders="true"/>
        <Button variant="secondary" on:click={testConnection}>
            Test connection <Icon iconName={connectionIcon}/>
        </Button>
        {#if correctAddress === false}
            <p class="error-text">Can't connect to this address :( is the server running okay?</p>
        {/if}
    </div>

    <hr class="mt-xsmall" style="border-bottom: none;">
    <label>Options</label>

    <div class="flex column">
        <Checkbox bind:checked={copyToClipboard}>Copy to clipboard</Checkbox>
        <Checkbox bind:checked={resetDecollisionMap}>Reset list of used identifiers</Checkbox>
    </div>
    <hr class="mt-xsmall" style="border-bottom: none;">

    <Textarea placeholder="Will output here" bind:value={generatedOutput}/>
    <div class="flex row-reverse">
        {#if disabled}
            Select a node to start
        {:else}
            Ready!
        {/if}
    </div>

    <div class="buttons flex justify-content-between ">
        <Button on:click={close} variant="secondary" class="mr-xsmall">Close</Button>
        <Button on:click={generatePressed} bind:disabled={disabled}>Generate</Button>
    </div>

</div>


<style>

    /* Add additional global or scoped styles here */
    .error-text {
        font-size: xx-small;
        color: indianred;
    }
</style>
