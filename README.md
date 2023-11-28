Original Author https://github.com/roshank231/onePaceMagnetExtractor
Modified to pull torrents instead of magnets

# onePaceTorrentExtractor

A script to extract torrent links from the OnePace site.

## Overview

This script allows users to easily gather torrent links from the [OnePace website](https://onepace.net/watch) without manually searching for each episode's link.

## Requirements

- A modern web browser with developer tools (e.g. Google Chrome, Mozilla Firefox)

## Usage

1. Visit the [OnePace site](https://onepace.net/watch).
2. Navigate to "Watch One Pace Episodes".
3. Right-click anywhere on the page and select `Inspect` or `Inspect Element` from the context menu to open the developer console.
4. In the developer console, switch to the `Console` tab.
5. Copy the entire code block from this repository and paste it into the console.
6. Hit `Enter` to run the script.
7. Follow the on-screen prompts to extract torrent links for your desired arc.

## Video Guide
https://github.com/roshank231/onePaceTorrentExtractor/assets/83808600/d3d552c7-5fdb-4028-bb45-420210492652

## Code

Here's the script you'll need:

```javascript
// Run in the console of the OnePace site at https://onepace.net/watch
// Ensure that no episodes are already open.

let arcsData = [];
let arcSections = document.querySelectorAll('section.Carousel_root__t7h0u');
let cumulativeEpisodes = 0;

arcSections.forEach(section => {
    let arcName = section.querySelector('h2 > span > div').innerText;
    let episodeElements = section.querySelectorAll('.CarouselSliderItem_item__fbsws');
    let episodesCount = episodeElements.length;

    arcsData.push({
        name: arcName,
        start: cumulativeEpisodes + 1,
        end: cumulativeEpisodes + episodesCount
    });

    cumulativeEpisodes += episodesCount;
});

function findArcByName(input) {
    for (let arc of arcsData) {
        if (arc.name.toLowerCase().includes(input.toLowerCase())) {
            return arc;
        }
    }
    return null;  // if no arc matches the input
}

function switchEpisode(episodeNumber, scrollToEpisode = true) {
    let episodeItems = document.querySelectorAll('.CarouselSliderItem_item__fbsws');

    if (episodeNumber > episodeItems.length || episodeNumber < 1) {
        console.error("Invalid episode number");
        return;
    }

    let selectedEpisode = episodeItems[episodeNumber - 1];

    // Scroll the selected episode into view only if scrollToEpisode is true
    if (scrollToEpisode) {
        selectedEpisode.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }

    selectedEpisode.click();
}

function getTorrentLink() {
    let links = Array.from(document.querySelectorAll('.Carousel_container__bEyKv > .Carousel_expander__FQ9Fs > .rmd-collapse > .Carousel_infoContainer__XQMVP > .Carousel_buttons__GB2gF [href^="https://api.onepace.net/download/torrent.php"]')).map(a => a.href);

    return links;
}

function gatherTorrentLinks(start, end) {
    let complete = [];
    let intervalTime = 200;  // 200ms or 0.20 seconds -- EDIT THIS DELAY TO BE HIGHER IS HAVING ISSUES!!!

    let episode = start;
    switchEpisode(episode);  // Scroll to the first episode

    let interval = setInterval(() => {
        let links = getTorrentLink();

        links.forEach(link => {
            if (!complete.includes(link)) {
                complete.push(link);
            }
        });

        if (complete.length > 0 || episode >= end) {
            episode++;

            if (episode <= end) {
                switchEpisode(episode, false);
            } else {
                clearInterval(interval);

                // Deselect the last episode
                let lastEpisode = document.querySelectorAll('.CarouselSliderItem_item__fbsws')[episode - 2];
                if (lastEpisode) lastEpisode.click();

                // Scroll back to top
                window.scrollTo({ top: 0, behavior: 'smooth' });

                setTimeout(() => {
                    console.log(`Detected ${complete.length} unique torrent links.`);
                    console.log("Completed gathering links. Here they are:");
                    console.log(complete.join('\n'));
                    console.log("Right click and 'Save As' the above links in the console to save as a TXT or LOG file to easily paste elsewhere!");
                    console.log("If the Links detected is less than expected, some/all episodes may already be batched into one link.");

                    // Ask to rerun the process
                    if (confirm("Do you want to rerun the script to scrape another arc?")) {
                        initScrapingProcess();
                    }
                }, 1000); // This delay ensures that the previous tasks complete before showing the prompt.
            }
        }
    }, intervalTime);
}

function initScrapingProcess() {
    // Gather user inputs
    let arcName = prompt("Enter the name of the arc:");
    let arc = findArcByName(arcName);

    if (arc) {
        console.log("Scraping Torrent Links for: " + arcName + '\n' + "...");
        gatherTorrentLinks(arc.start, arc.end);
    } else {
        console.error("Arc not found. Please check the name and try again.");
    }
}

// Start the process
initScrapingProcess();
