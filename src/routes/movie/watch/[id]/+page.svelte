<script lang="ts">
	import Videoplayer from "$lib/components/videoplayer.svelte";
	import type { Movie, Cast } from "$lib/types/movie";
	import { PlayCircle, Star, Clock, Tv } from "lucide-svelte";
	import Carousel from "$lib/components/carousel.svelte";
  import '@splidejs/svelte-splide/css';
	import { Splide, SplideSlide } from "@splidejs/svelte-splide";
	import { options } from "$lib/config/carousel";

  export let data: {
    details?: Movie;
    recommendations: Movie[];
    similar: Movie[];
    casts: Cast[];
  };

  let { details, recommendations, similar, casts } = data;

  let watchData: Movie | undefined;
  let isLoading = true;

  const fetchData = async () => {
    isLoading = true;
    const res = await fetch(`${import.meta.env.VITE_DETAILS_URL}${details?.id}?type=movie`);
    watchData = await res.json();
    isLoading = false;
  };

  const formatRuntime = (runtime: number | undefined) => {
    if (runtime === undefined) {
      return 'N/A';
    }
    const hours = Math.floor(runtime / 60);
    const minutes = runtime % 60;
    return `${hours}h ${minutes}m`;
  };

  $: {
    ({ details, recommendations, similar, casts } = data);
    fetchData();
  }
</script>

{#if details}
  <section class="container grid items-center">
    {#if watchData}
      <Videoplayer 
        url={`${import.meta.env.VITE_WATCH_URL}${watchData.episodeId}?id=${watchData.id}`}
      />
    {:else}
      <div class="w-full h-60 sm:h-auto md:h-[50vh] lg:h-[60vh] xl:h-[70vh] relative">
        {#if !isLoading}
          <img
            class="w-full h-full object-cover rounded"
            src={`https://image.tmdb.org/t/p/w500/${details.backdrop_path}`}
            alt={details.title}
          />
          <div class="absolute top-0 left-0 w-full h-full" style="background: linear-gradient(to bottom, rgba(0, 0, 0, 0.6), rgba(0, 0, 0, 0.6));"></div>
        {:else}
          <div class="w-full h-full bg-black"></div>
        {/if}
      </div>
    {/if}
  </section>
{/if}

<div class="container grid items-center gap-4 pb-8 pt-6 md:py-2 relative">
  <h1 class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl text-primary">{details?.title || 'N/A'}</h1>
  <span class="flex items-center">
    <Tv class="h-4 w-4 mr-2 text-primary"/>
    MOVIE
    <Star class="h-4 w-4 mx-2 text-primary"/>
    {details?.vote_average.toFixed(1)}
    <Clock class="h-4 w-4 mx-2 text-primary"/>
    {formatRuntime(details?.runtime)}
  </span>
  <blockquote class="italic">
    {details?.tagline || ''}
  </blockquote>
  <p>{details?.overview || ''}</p>
  <p>Released: {details?.release_date}</p>
  <p>Genres: {details?.genres.map(genre => genre.name).join(', ')}</p>
  <p>Casts: {casts.slice(0, 5).map(cast => cast.name).join(', ')}</p>
</div>

{#if ((recommendations && recommendations.length > 0) || (similar && similar.length > 0))}
  <div class="container grid items-center gap-4 pb-8 pt-6 md:py-2 relative mt-8">
    <h3 class="scroll-m-20 text-2xl text-primary font-semibold tracking-tight mb-2">
      You may also like:
    </h3>
    {#if recommendations && recommendations.length > 0}
      <Carousel items={recommendations} url="/movie" title="title" />
    {:else}
      <Carousel items={similar} url="/movie" title="title" />
    {/if}
  </div>
{/if}