# Sketches

<style>
    .sketch-container {
        display: grid;
        grid-template-columns: 1fr;
        gap: 0.25rem;
    }

    .sketch-item img {
        width: 100%;
        height: auto;
    }

    @media only screen and (min-width: 600px) {
        .sketch-container {
            grid-template-columns: repeat(2, 1fr);
        }

        .sketch-item {
            width: 100%;
            background-color: #cfcfcf;
            display: flex;
            align-items: center;
        }
    }

    @media only screen and (min-width: 768px) {
        .sketch-container {
            grid-template-columns: repeat(3, 1fr);
        }

        .sketch-item {
            width: 100%;
            background-color: #cfcfcf;
            display: flex;
            align-items: center;
        }
    }
</style>

<div class="sketch-container">
    <div class="sketch-item"><img src="./sketch/Untitled6.jpg" alt="musashi 2"></div>
    <div class="sketch-item"><img src="./sketch/Untitled5.jpg" alt="witcher"></div>
    <div class="sketch-item"><img src="./sketch/Untitled4.jpg" alt="tokyo ghoul 2"></div>
    <div class="sketch-item"><img src="./sketch/Untitled3.jpg" alt="weeknd"></div>
    <div class="sketch-item"><img src="./sketch/Untitled2.jpg" alt="musashi"></div>
    <div class="sketch-item"><img src="./sketch/Untitled1.png" alt="tokyo ghoul"></div>
    <div class="sketch-item"><img src="./sketch/Untitled.jpg" alt="jack jeanne"></div>
</div>