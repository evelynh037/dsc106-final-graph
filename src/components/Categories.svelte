<script>
    import { onMount } from 'svelte';
    import { scaleBand, scaleLinear } from 'd3-scale';
	import * as d3 from 'd3';
    export let index, selectedCategory ;
    let width = 500;
    let height = 500;
	let categories = [];
    let xTicks = [];
    let yTicks = [];
	let isMounted = false;
    const padding = { top:100, right: 10, bottom: 30, left: 20 };
    let filteredData = [];
	let tooltip = { text: "",text2: "",text2:"", visibility: 'hidden', x: 0, y: 0  };
	onMount(async () => {
		const res = await fetch('category_sum_all.csv');
		const csv = await res.text();
		categories = d3.csvParse(csv, d3.autoType)
		updateChart(selectedCategory);
	})

	onMount(() => {
		// Set isMounted to true after the component has been mounted
		isMounted = true;
	});

    function updateChart(selectedCategory) {
        filteredData = categories.filter(
        (d) => d.topic === selectedCategory
        );
        xTicks = filteredData.map((d) => d.category);
    }


    $: maxFrequency = Math.max(...filteredData.map(d => d.difference));

    function generateYTicks(maxFrequency, step) {
        const ticks = [];
        for (let i = 0; i <= maxFrequency; i += step) {
            ticks.push(i);
        }
        return ticks;
    }

    $: yTicks = generateYTicks(maxFrequency, maxFrequency / 4);

    $: xScale = scaleBand()
        .domain(xTicks)
        .range([padding.left, width + padding.right])
        .padding(0.5);

    $: yScale = scaleLinear()
        .domain([0, maxFrequency])
        .range([height - padding.bottom, padding.top]);

	function showTP(bar, event) {
		if (bar.dt_more == 1){
			tooltip.text = "Out of his 3000 tweets, Trump"
			tooltip.text2 = "brought up " + bar.category
			tooltip.text3 = String(bar.difference) + " more times than Clinton"
		}
		else{
			tooltip.text = "Out of her 3000 tweets, Clinton"
			tooltip.text2 = "brought up " + bar.category
			tooltip.text3 = String(bar.difference) + " more times than Trump"
		}
        const barWidth = xScale.bandwidth();
        const xCoordinate = xScale(bar.category) + barWidth / 2;
        const yCoordinate = yScale(bar.difference)-50;
        const isRightHalf = xCoordinate > width / 2;
        if (isRightHalf) {
            tooltip.x = xCoordinate - 110
        } else {
            tooltip.x = xCoordinate +5;
        }

        tooltip.y = yCoordinate;
        tooltip.visibility = 'visible';
}

    function hideTP() {
        tooltip.visibility = 'hidden';
    }
</script>


{#if (index === 5 || index === 6) && isMounted} 
<div class = 'fade-in'>
	<h2>Difference in Number of Tweets in Different Topics</h2>
	<div style="margin-left: 50px;">
		<label for="categorySelect" style="font-size: 1.0em;">Select Category:</label>
		<select id="categorySelect" style="font-size: 0.9em; height: 2em; width: 15em;" bind:value={selectedCategory} on:change="{() => updateChart(selectedCategory)}">
		<option>demographic_issue</option>
		<option>social_issue</option>
		<option>security_policy</option>
		<option>emotional_word</option>
		<option>economic_policy</option>
		<option>campaign_message</option>
		<option>national_identity</option>
		</select>
	</div>
	<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
		<svg>
			<!-- Legend -->
			<g class="legend" transform="translate({width-130}, {padding.top - 60})">
				<rect x="0" y="0" width="15" height="15" fill="#ff4500" />
				<text x="30" y="12" font-size="0.9em">Trump has more</text>
				<rect x="0" y="20" width="15" height="15" fill="#00bfff" />
				<text x="30" y="32" font-size="0.9em">Clinton has more</text>
			</g>

			<!-- y axis -->
			<g class="axis y-axis">
				{#each yTicks as tick}
					<g class="tick tick-{tick}" transform="translate(0, {yScale(tick)})">
						<line x2="100%" />
						<text y="-4">{tick} {tick === maxFrequency ? ' per 3,000 tweets' : ''}</text>
					</g>
				{/each}
			</g>

			<!-- x axis -->
			<g class="axis x-axis">
				{#each filteredData as point, i (i)}
					<g class="tick" transform={`translate(${xScale(point.category) + xScale.bandwidth()}, ${height - padding.bottom -9})`}>
						<text transform="rotate(0)" x="-9"  dy="1.5em"  text-anchor="middle">
							{point.category}
						</text>
					</g>
				{/each}
			</g>

			<g class="bars">
				{#each filteredData as point, i (i)}
					<rect
						class="barGrow"
						x={xScale(point.category)}
						y={yScale(point.difference)}
						width={xScale.bandwidth() * 1.7}
						height={yScale(0) - yScale(point.difference)}
						fill={point.hc_more === 1 ? '#00bfff' : '#ff4500'}
						on:mouseover={(event) => showTP(point, event)}
						on:mousemove={(event) => showTP(point, event)}
						on:mouseout={hideTP}>
					</rect>

				{/each}
			</g>
			<g class="tooltip-box" transform={`translate(${tooltip.x}, ${tooltip.y})`} visibility={tooltip.visibility}>
				<rect x={-10} y={-30} width={190} height={50} fill="#D3D3D3" stroke="grey" />
				<text x={10} y={-15} font-size="0.8em">{tooltip.text}</text>
				<text x={10} y={0} font-size="0.8em">{tooltip.text2}</text>
				<text x={10} y={15} font-size="0.8em">{tooltip.text3}</text>
			</g>	  
		</svg>
	</div>
  </div>
{/if}

<style>

	h2 {
		text-align: center;
		font-size: 1.7em;
	}

	.chart {
	display: flex;
    justify-content: center;
    align-items: center;
    width: 60%; 
    margin: auto;
	}

	svg {
		width: 100%;
		height: 440px;
	}

	.y-axis .tick text {
		font-family: Helvetica, Arial;
		font-size: 0.9em;
	}

	.tick line {
		stroke: #525151;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: 'black';
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.x-axis .tick text {
		text-anchor: middle;
		font-size: 0.85em;
	}

	.bars rect {
		stroke: none;
		opacity: 0.9;
	}
	@keyframes barGrow {
		from {
			height: 0;
		}
	}

	.barGrow {
		animation: barGrow 1s ;
	}
	.tooltip {
		font-size: 14px;
		fill: rgb(2, 100, 12);
		font-weight: bold;
	}

	@keyframes fadeInAnimation {
		from {
		opacity: 0;
		}
		to {
		opacity: 1;
		}
	}

	/* Apply fade-in animation only when component is mounted */
	.fade-in {
		opacity: 0;
		animation: fadeInAnimation 1s ease-in forwards;
	}

	/* Delay fade-in animation until component is mounted */
	.fade-in-delayed {
		opacity: 0;
		animation: fadeInAnimation 1s ease-in forwards;
	}
</style>