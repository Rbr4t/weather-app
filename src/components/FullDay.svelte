<script>
    export let sky;
    export let time;
    export let temp;

</script>

<li class="py-3 sm:py-4">
    <div class="flex items-center space-x-4 justify-between">
        <div class="flex-shrink-0">
          <h3 class="font-semibold text-xl">{time}</h3>
          
        </div>
        
        <div class="flex gap-5">
          <div class="flex">
            
            <img src={sky} alt="" class="w-8 p-0">
            <div class="inline-flex items-center text-base font-semibold text-gray-900 dark:text-white">
              {temp}°C
            </div>
          </div>
          
          
        </div>
        
    </div>
</li>