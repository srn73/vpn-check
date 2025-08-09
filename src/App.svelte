<script lang="ts">
	import { onMount } from "svelte";

    let requestFailed: boolean = $state(false);
    let data: IPApiResponse | undefined = $state(undefined);

    interface IPApiResponse {
        "status": "success" | "fail",
        "country": string,
        "region": string,
        "regionName": string,
        "city": string,
        "zip": string,
        "lat": number,
        "lon": number,
        "timezone": string,
        "isp": string,
        "org": string,
        "as": string,
        "proxy": boolean,
        "query": string
    };

    onMount(async () => {
        const apiResponse = await fetch("https://ip-api.com/json/?fields=status,message,country,region,regionName,city,zip,lat,lon,timezone,isp,org,as,proxy,query");

        if (apiResponse.ok) {
            data = await apiResponse.json();
            if (data.status == "fail") requestFailed = true;
        } else {
            requestFailed = true;
        };
    });
</script>

<main class="p-4">
    {#if requestFailed == false}
        <h2>You are <span class="{data?.proxy == true ? "connected" : "disconnected"}">{data?.proxy != undefined ? data?.proxy == true ? "connected" : "not connected" : "..."}</span> to a VPN.</h2>
        
        <p>IP data provided by IP-API.com</p>

        <table class="table">
            <thead>
                <tr>
                    <th scope="col">Name</th>
                    <th scope="col">Value</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <th scope="row">VPN / Proxy</th>
                    <td>{data?.proxy == true ? "✅" : "❌"}</td>
                </tr>
                <tr>
                    <th scope="row">IP Address</th>
                    <td>{data?.query}</td>
                </tr>
                <tr>
                    <th scope="row">Service</th>
                    <td>{data?.org || data?.isp || "No provider found"}</td>
                </tr>
                <tr>
                    <th scope="row">Location</th>
                    <td>{data?.city || "[city]"}, {data?.regionName || "[region]"}</td>
                </tr>
                <tr>
                    <th scope="row">Coordinates</th>
                    <td>{data?.lat}, {data?.lon}</td>
                </tr>
            </tbody>
        </table>

        {#if data?.lat && data?.lon}
            <p>Map provided by OpenStreetMap. Keep in mind, this is an estimation and isn't the exact location of this IP address.</p>
            <iframe title="OpenStreetMap embed" width="500px" height="350px" src="https://www.openstreetmap.org/export/embed.html?bbox={data.lon - 0.025},{data.lat - 0.025},{data.lon + 0.025},{data.lat + 0.025}&layer=mapnik&marker={data.lat},{data.lon}" style="border: 1px solid black"></iframe>
        {:else}
            <p>No estimated location was found for this address.</p>
        {/if}
    {:else}
        <h2>The request to IP-API failed. (most likely due to rate limiting)</h2>
    {/if}
</main>
