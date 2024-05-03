<div class="index-container">

<div class="index-sidebar">
I do some programming in Go and I like doing system administration too. Sometimes I get sad and binge eat my <a href="./like-food.html">favorite food</a>. Also, I <a href="./sketches.html">sketch</a>, usually panels from my favorite comics.<br><br>

<div style="position: relative; display: inline-block;">
    <img src="./assets/images/my_stuff.png" alt="Alt Text" style="width: 100%;">

    <style>
        .smallsc {
            position: absolute; 
            bottom: 25%; 
            left: 25%;
            list-style: none; 
            padding: 0; 
            margin: 0; 
            background-color: rgba(0, 0, 0, 0.5); 
            padding: 10px; 
        }

        @media (max-width: 600px) {
            .smallsc {
                bottom: 10%;
                left: 0%;
            }
        }
    </style>

    <div class="smallsc">
        <a href="mailto:mprasadme@gmail.com" style="color: white;">Gmail</a><br>
        <a href="https://github.com/snwzt" style="color: white;">GitHub</a><br>
        <a href="https://www.linkedin.com/in/mdehury" style="color: white;">LinkedIn</a><br>
        <a href="https://twitter.com/sloflayer" style="color: white;">Twitter</a><br>
        <a href="./sketches.html" style="color: white;">Sketches</a><br>
    </div>
</div>

</div>

<div class="index-main-content">

<h1>Blogs</h1>

<ul>
    <li><a href="./l4-l7-lb.html">Load Balancer vs Application Load Balancer</a></li>
    <li><a href="./ptr-go.html">Pointers in Go</a></li>
    <li><a href="./like-food.html">Food I love</a></li>
    <li><a href="./multi-stage-oci.html">Multi-stage OCI builds</a></li>
    <li><a href="./basic-regex.html">Basic Regex</a></li>
    <li><a href="./osi-model-oversimplified.html">OSI Model Oversimplified</a></li>
</ul>

</div>

</div>

<style>
.index-container {
    display: flex;
    flex-direction: row;
}

.index-sidebar {
    flex: 1;
    margin-right: 20px;
}

.index-main-content {
    flex: 1;
}

@media (max-width: 1000px) {
    .index-container {
        flex-direction: column;
    }

    .index-sidebar {
        margin-right: 0;
    }
}

</style>