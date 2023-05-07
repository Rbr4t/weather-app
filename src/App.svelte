<script>
  import FullDayCard from './components/FullDay.svelte';
  import WeekCard from './components/Week.svelte';

  let activeTab = 'week-tab'
  let selected_city = 'London'
  let error;
  const handleTabChange = (prop) => {
    activeTab = prop;
  }

  

  async function fetchCurrentWeather(param){
    let desired_outcome = null
    await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${import.meta.env.VITE_API_KEY}&q=${selected_city}&days=1&aqi=yes&alerts=no`, {mode: 'cors'})
      .then((resp) => {return resp.json()})
      .then((obj) => {
        switch(param){
          case 'temp':
            desired_outcome = obj.current.temp_c;
            break
          case 'rain':
            desired_outcome = obj.forecast.forecastday[0].day.daily_chance_of_rain;
            break
          case 'humidity':
            desired_outcome = obj.current.humidity;
            break
          case 'air':
            desired_outcome = obj.current.air_quality["gb-defra-index"];
            break
          case 'wind':
            desired_outcome =  obj.current.wind_kph
            break;
          case 'condition':
          desired_outcome = {'text': obj.current.condition.text, 'icon': obj.current.condition.icon}
        }
        
      })
      return desired_outcome
  }
  
  async function fetchWeatherByHours() {
    let byHours = [];
    let currentTime = new Date().getHours();
    let day = 0
    
    await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${import.meta.env.VITE_API_KEY}&q=${selected_city}&days=2&aqi=yes&alerts=no`, {mode: 'cors'})
      .then((resp) => {return resp.json()})
      .then((d) => {
        console.log(d)
        for(let i=0; i<12; i++){
          const temp_c = d.forecast.forecastday[day].hour[currentTime].temp_c;
          
          const new_entry = {"temp": temp_c, "time": `${currentTime}:00`, 'sky':d.forecast.forecastday[day].hour[currentTime].condition.icon} // TODO: Add more properties
          byHours.push(new_entry)
          currentTime += 1
          if(currentTime == 24) {
            currentTime = 0
            day = 1
          }
        }
      })
    
    return byHours
  }

  async function fetchWeatherForecast5() {
    let forecast = []
    await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${import.meta.env.VITE_API_KEY}&q=${selected_city}&days=6&aqi=yes&alerts=no`, {mode: 'cors'})
      .then((resp) => {return resp.json()})
      .then((d) => {
        for(let i=1; i<2; i++){
          
          const obj = d.forecast.forecastday[i]
          const date = new Date(obj.date);
          const avg_temp = obj.day.avgtemp_c
          const rain_chance = obj.day.daily_chance_of_rain
          const icon = obj.day.condition.icon
          let weekday;
          
          switch(date.getDay()){
            case 1:
              weekday = 'Monday';
              break
            case 2:
              weekday = 'Tuesday';
              break
            case 3:
              weekday = 'Wednesday';
              break
            case 4:
              weekday = 'Thursday';
              break
            case 5:
              weekday = 'Friday';
              break
            case 6:
              weekday = 'Saturday';
              break
            case 0:
              weekday = 'Sunday';
              break

          }
          forecast.push({"date":`${date.getDate()}.${date.getMonth()+1}`, avg_temp, rain_chance, icon, weekday}) 
        }
      })
    return forecast
  }

  

  let temp_c = fetchCurrentWeather('temp') 
  let rain = fetchCurrentWeather('rain')
  let humidity = fetchCurrentWeather('humidity')
  let wind = fetchCurrentWeather('wind')
  let air_quality = fetchCurrentWeather('air')
  let current_weather = fetchCurrentWeather('condition')
  
  let forecast5 = fetchWeatherForecast5()
  let forecast12 = fetchWeatherByHours()

  const updateWeather = async() => {
    let everythingOk = false
    error = true // Has a bug of for a moment showing an error with the input, can be solved better but I'm too lazy.
    await fetch(`https://api.weatherapi.com/v1/current.json?key=${import.meta.env.VITE_API_KEY}&q=${selected_city}`, {mode: 'cors'})
      .then((resp) => {
        console.log(resp.ok)
        if(resp.ok){
          everythingOk = resp.ok
        
          
          error = false
        
          
          

        }
      })
    console.log(error)
      
        
      
    if(everythingOk){
      temp_c = fetchCurrentWeather('temp') 
      rain = fetchCurrentWeather('rain')
      humidity = fetchCurrentWeather('humidity')
      wind = fetchCurrentWeather('wind')
      current_weather = fetchCurrentWeather('condition')
      air_quality = fetchCurrentWeather('air')
      
      forecast5 = fetchWeatherForecast5()
      forecast12 = fetchWeatherByHours()
    }
    
    
  } 
  
  
</script>

<main class="grid sm:grid-cols-1 lg:grid-cols-3 p-4 grid-rows-[min-content,1fr,min-content] bg-slate-50 bg-cover max-w-full min-h-screen bg-no-repeat " style="background-image: url(src/assets/wonderful.jpg)">
  
  <header class="opacity-70 flex p-8 align-middle bg-white col-span-3 justify-between max-[768px]:flex-col max-[768px]:gap-4 md:flex-row rounded-lg items-center min-h-min"> 
    <h1 class="p-0 m-0 text-5xl font-extrabold dark:text-white">Weather App</h1>

    <div class=" place-content-stretch flex justify-center">
    
      
        <div class="flex items-center gap-2">
          
          <label for="selected_city" class="block mb-2 text-lg font-medium text-gray-900 dark:text-white whitespace-nowrap">Select city:</label>
          {#if error}
          
          <div>
            <input bind:value={selected_city} type="text" id="selected_city" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"required>
            <p class="mt-2 text-sm text-red-600 dark:text-red-500">City not found.</p>
          
          </div>
          {:else}
            <input bind:value={selected_city} type="text" id="selected_city" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"required>

          {/if}
          
          
          <button on:click={updateWeather} type="button" ><img src="src/assets/search.svg" alt="" class="w-10 h-10  "></button>
          
        </div>
        
        
      
  
    </div>
  </header>

 
  <div class="opacity-85 w-full m-4 max-w-md p-4 bg-white border border-gray-200 rounded-lg shadow sm:p-8 dark:bg-gray-800 dark:border-gray-700 row-start-2 row-span-1 justify-self-center">
    <ul class="flex flex-wrap text-sm font-medium text-center text-gray-500 border-b border-gray-200 rounded-t-lg bg-gray-50 dark:border-gray-700 dark:text-gray-400 dark:bg-gray-800" id="defaultTab" data-tabs-toggle="#defaultTabContent" role="tablist">
      <li class="mr-2">
          <button on:click={() => handleTabChange('week-tab')} id="week-tab" class:selected="{activeTab==='week-tab'}" class="inline-block p-4 rounded-tl-lg hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-blue-500">5 day</button>
      </li>
      <li class="mr-2">
          <button on:click={() => handleTabChange('12h-tab')} id="12h-tab" class:selected="{activeTab==='12h-tab'}" class="inline-block p-4 hover:text-gray-600 hover:bg-gray-100 dark:hover:bg-gray-700 dark:hover:text-gray-300">12 hour</button>
      </li>
      <li class="mr-2">
          <button on:click={() => handleTabChange('air-tab')} id="air-tab" class:selected="{activeTab==='air-tab'}" class="inline-block p-4 hover:text-gray-600 hover:bg-gray-100 dark:hover:bg-gray-700 dark:hover:text-gray-300">Air quality</button>
      </li>
  </ul>
  
    

  {#if activeTab=='week-tab'}
    <!-- tab 1 -->
   <div class="flow-root " id="week-tab" role="tabpanel" aria-labelledby="week-tab">
        <ul role="list" class="divide-y divide-gray-200 dark:divide-gray-700">
          {#await forecast5}
            <div role="status" class="flex justify-center items-center h-full">
              <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
                  <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
              </svg>
              <span class="sr-only">Loading...</span>
          </div>
          {:then data} 
            {#each data as d}
              <WeekCard sky={d.icon} date={d.date} weekday={d.weekday} temp={d.avg_temp} rainchance={d.rain_chance}/>
            {/each}
          {/await}
            
            
        </ul>
   </div>
   <!-- tab 2 -->
   {:else if activeTab=='12h-tab'}
    <div class="flow-root overflow-y-auto max-h-96" id="12h-tab" role="tabpanel" aria-labelledby="week-tab">
      <div class="h-fit" id="week-tab" role="tabpanel" aria-labelledby="week-tab">
        <ul role="list" class="p-3 divide-gray-200 dark:divide-gray-700 max-h-min">
            {#await forecast12}
              <div role="status">
                <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
                    <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
                </svg>
                <span class="sr-only">Loading...</span>
            </div>
            {:then obj} 
              {#each obj as data}
                <FullDayCard time={data.time} temp={data.temp} sky={data.sky}/>
              {/each}
            {/await}
            
            
        </ul>
   </div>
    </div>
   <!-- tab 3 -->
   {:else if activeTab=='air-tab'}
    <div class="flex justify-center items-center pt-8 " id="about" role="tabpanel" aria-labelledby="week-tab">
      <div class="text-4xl w-64 h-64 border rounded-full flex items-center justify-center border-8 border-none bg-gradient-to-r from-sky-300 to-indigo-500">
        {#await air_quality}
          <div role="status">
    <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <span class="sr-only">Loading...</span>
</div>
        {:then number} 
          Index: {number}
        {/await}
      </div>
      
    </div>
  {/if}

</div>

  <div id="weather-details" class="p-9 flex gap-5 row-start-3 flex-wrap col-span-full justify-around content-center ">
    
    <div class="opacity-70 max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow dark:bg-gray-800 dark:border-gray-700 flex items-center gap-4">
      {#await current_weather}
      <div role="status">
        <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
            <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
        </svg>
        <span class="sr-only">Loading...</span>
    </div>
      {:then stuff} 
        <img src={stuff.icon} class=" w-16 ">
        <h5 class=" mb-2 text-2xl font-semibold tracking-tight text-gray-900 dark:text-white "> {stuff.text}</h5>
      {/await}
      
    </div>

    <div class="opacity-70 max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow dark:bg-gray-800 dark:border-gray-700 flex items-center">
        <img src="src/assets/temperature.svg" class="w-16 ">
        {#await temp_c}
          <div role="status">
    <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <span class="sr-only">Loading...</span>
</div>
        {:then number}
          <h5 class="mb-2 text-2xl font-semibold tracking-tight text-gray-900 dark:text-white ">{number}°C</h5>
        {/await}
   </div>

    <div class="opacity-70 max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow dark:bg-gray-800 dark:border-gray-700 flex items-center">
      <img src="src/assets/rain.svg" class="w-16 ">
      {#await rain}
        <div class="w-full bg-gray-200 rounded-full h-2.5 dark:bg-gray-700">
          <div class="bg-blue-600 h-2.5 rounded-full" style="width: 45%"></div>
        </div>
        {:then number}
          <h5 class="mb-2 text-2xl font-semibold tracking-tight text-gray-900 dark:text-white ">{number}%</h5>
        {/await}
    </div>

    
    <div class="opacity-70 max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow dark:bg-gray-800 dark:border-gray-700 flex items-center">
      <img src="src/assets/waterdroplet.svg" class="w-16 ">
      {#await humidity}
        <div class="w-full bg-gray-200 rounded-full h-2.5 dark:bg-gray-700">
          <div class="bg-blue-600 h-2.5 rounded-full" style="width: 45%"></div>
        </div>
        {:then number}
          <h5 class="mb-2 text-2xl font-semibold tracking-tight text-gray-900 dark:text-white ">{number}%</h5>
        {/await}
    </div>
    
    <div class="opacity-70 max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow dark:bg-gray-800 dark:border-gray-700 flex items-center">
      <img src="src/assets/windspeed.svg" class="w-16 ">
        {#await wind}
          <div role="status">
    <svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <span class="sr-only">Loading...</span>
</div>
        {:then number}
          <h5 class="mb-2 text-2xl font-semibold tracking-tight text-gray-900 dark:text-white ">{Math.round(number*1000/3600*100)/100} m/s</h5>
        {/await}
    </div>
    
  
  </div>
  

  
  
</main>

<style>
  .selected {
    color: rgb(37 99 235 / var(--tw-text-opacity));
  }

  html { height: 100%; }
  /* :global(body){
        background-color: lightseagreen;
        background-image: url("src/assets/bg.jpg");
        
        background-repeat: no-repeat;
    } 
  */
  body { min-height: 100%; }
</style>
