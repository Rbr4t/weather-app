<script>
    export let sky;
    export let date;
    export let weekday;
    export let temp;
    export let rainchance;
</script>

<li class="py-3 sm:py-4">
    <div class="flex items-center space-x-4 justify-between">
        <div class="flex-shrink-0">
          <h3 class="font-semibold text-xl">{date}</h3>
          <h5>{weekday}</h5>
        </div>
        
        <div class="flex gap-5">
        <div>
            
            <img src={sky} alt="" class="w-8 p-0">
        </div>

          <div class="flex">
            <img src="src/assets/temperature.svg" alt="" class="w-8 p-0">
            <div class="inline-flex items-center text-base font-semibold text-gray-900 dark:text-white">
              {temp}°C
            </div>
          </div>
          <div class="flex">
            <img src="src/assets/rain.svg" alt="" class="w-8 p-0">
            <div class="inline-flex items-center text-base font-semibold text-gray-900 dark:text-white">
              {rainchance}%
              </div>
          </div>
          
        </div>
        
    </div>
</li>