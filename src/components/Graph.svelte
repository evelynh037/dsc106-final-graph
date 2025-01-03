<script>
  import { scaleBand, scaleLinear } from 'd3-scale';
  export let index, filtered_trump, filtered_hillary, selected_word;
  import { onMount } from 'svelte';

  let isMounted = false;
  let width = 500;
  let height = 300;
  const padding = { top: 70, right: 50, bottom: 30, left: 50 };

  onMount(() => {
		// Set isMounted to true after the component has been mounted
		isMounted = true;
	});

  // Combine the two and assign with category for bars
  $: combinedData = [
    ...filtered_hillary.map(d => ({ ...d[0], category: 'Hillary' })),
    ...filtered_trump.map(d => ({ ...d[0], category: 'Trump' }))
  ];

  // Get proportion
  $: selected_word.forEach((element) => {
    let filtered = combinedData.filter((d) => d.Word == element.toLowerCase())
    let total = filtered.reduce((acc, item) => acc + item.Frequency, 0)
    filtered.forEach((element) => {
      element.proportion = (element.Frequency / total) * 100
    });
  });

  // Create scales
  $: yScale = scaleBand()
    .domain(selected_word)
    .range([padding.top, height - padding.bottom])
    .padding(0.1);

  $: xScale = scaleLinear()
    .domain([0, 100])
    .range([padding.left, width - padding.right]);

  // Generate x-axis ticks
  function generateXTicks(maxFrequency, step) {
    const ticks = [];
    for (let i = 0; i <= maxFrequency; i += step) {
      ticks.push(i);
    }
    return ticks;
  }
  $: xTicks = generateXTicks(100, 100 / 5);
</script>

{#if (index === 7 || index === 8) && isMounted} 
  <div class='fade-in'>
  <h2>Decomposition of Selected Words in Tweets from two Candidates </h2>
  <h4>Of all tweets posted by Clinton and Trump that contains the selected word, whose tweets takes larger proportion?</h4>
  <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
    <svg>
      <g class="legend" transform="translate({width-80}, {padding.top - 60})">
        <rect x="0" y="0" width="15" height="15" fill="#ff4500" />
        <text x="30" y="12" font-size="0.9em">Trump</text>
        <rect x="0" y="20" width="15" height="15" fill="#00bfff" />
        <text x="30" y="32" font-size="0.9em">Hillary</text>
        </g>
      <g class="axis x-axis" transform="translate(0, {padding.top})">
        {#each xTicks as tick}
          <g class="tick tick-{tick}" transform="translate({xScale(tick)}, 0)">
            <line y2="{height - padding.top - padding.bottom}" />
            <text x="{tick === 1 ? -1 : 1}" dy="1.2em" text-anchor="{tick === 0 ? 'end' : 'start'}">{tick}%</text>
          </g>
        {/each}
      </g>
      <g class="axis y-axis" transform="translate({padding.left}, 0)">
        {#each selected_word as word}
          <g class="tick" transform="translate(0, {yScale(word) + yScale.bandwidth() / 2})">
            <text transform="rotate(0)" x="3" y="{yScale.bandwidth()/4.25}"  dy="0.35em" >{word}</text>
          </g>
        {/each}
      </g>
      <g class="bars">
        {#each combinedData as {Word, proportion, category}, i}
          <rect
            class="grow"
            y={yScale(Word) + (category === 'Hillary' ? yScale.bandwidth() / 4 : yScale.bandwidth() / 4)}
            x={(category === 'Hillary' ? xScale(0) : xScale(100) - xScale(proportion) + padding.left)}
            height={yScale.bandwidth() / 2.5}
            width={xScale(proportion) - xScale(0)}
            fill={category === 'Hillary' ? '#00bfff' : '#ff4500'}
          />
        {/each}
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
h4 {
  text-align:center;
}

	.chart {
	display: flex;
    justify-content: center;
    align-items: center;
    width: 65%; 
    margin: auto;
	}

	svg {
		width: 120%;
		height: 500px;
	}

	.tick {
		font-family: "Times New Roman", Times, serif;
		font-size: 0.725em;
		font-weight: 200;
	}

	.tick line {
		stroke: #7e7d7d; 
    stroke-dasharray: 3;
	}

	.tick text {
		fill: #000000;
		text-anchor: start;
    font-size: 1.6em;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.bars rect {
		opacity: 0.9;
	}
  @keyframes grow {
    from {
        width: 0;
    }
  }

  .grow {
      animation: grow 1s ease-out;
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