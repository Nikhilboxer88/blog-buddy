# blog-buddy
**Blog Buddy AI** is a WordPress plugin designed to help bloggers and content creators generate high-quality blog titles, intros, and full posts using OpenAI's powerful language model. 

<?php
/*
Plugin Name: Blog Buddy AI
Description: Generate blog titles, intros, and full posts using OpenAI.
Version: 1.0
Author: Nikhil Dhaka
*/

add_action('admin_menu', 'blog_buddy_ai_menu');

function blog_buddy_ai_menu() {
    add_menu_page(
        'Blog Buddy AI',
        'Blog Buddy AI',
        'manage_options',
        'blog-buddy-ai',
        'blog_buddy_ai_dashboard'
    );
}

function blog_buddy_ai_dashboard() {
    ?>
    <div class="wrap">
        <h1>Blog Buddy AI</h1>
        <form id="blog-buddy-form">
            <input type="text" name="topic" id="topic" placeholder="Enter your topic" style="width:300px;" required />
            <br><br>
            <select id="type">
                <option value="title">Generate Title</option>
                <option value="intro">Generate Intro</option>
                <option value="full">Generate Full Post</option>
            </select>
            <br><br>
            <button type="button" class="button button-primary" onclick="generateContent()">Generate</button>
        </form>
        <br>
        <div id="result"></div>
    </div>
    <script>
        // Use unique variable name to avoid conflict
        var blogBuddyAjaxUrl = "<?php echo admin_url('admin-ajax.php'); ?>";

        // Define the generateContent function properly
        function generateContent() {
            var topic = document.getElementById('topic').value;
            var type = document.getElementById('type').value;

            document.getElementById('result').innerHTML = "<em>Generating... Please wait.</em>";

            fetch(blogBuddyAjaxUrl, {
                method: 'POST',
                headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                body: 'action=blog_buddy_generate&topic=' + encodeURIComponent(topic) + '&type=' + type
            })
            .then(response => response.text())
            .then(result => {
                document.getElementById('result').innerHTML = '<h3>AI Output:</h3><p>' + result + '</p>';
            })
            .catch(error => {
                document.getElementById('result').innerHTML = '<p style="color:red;">Error: ' + error + '</p>';
            });
        }
    </script>
    <?php
}

add_action('wp_ajax_blog_buddy_generate', 'blog_buddy_generate_content');

function blog_buddy_generate_content() {
    $topic = sanitize_text_field($_POST['topic']);
    $type = sanitize_text_field($_POST['type']);

    if (!$topic || !$type) {
        echo 'Missing topic or type.';
        wp_die();
    }

    $prompt = '';
    switch ($type) {
        case 'title':
            $prompt = "Generate a catchy blog post title about: $topic";
            break;
        case 'intro':
            $prompt = "Write an engaging introduction for a blog post about: $topic";
            break;
        case 'full':
            $prompt = "Write a full blog post of about 500 words on the topic: $topic";
            break;
    }

    $response = wp_remote_post('https://api.openai.com/v1/chat/completions', [
        'headers' => [
            'Authorization' => 'Bearer enter you api key here  ',
            'Content-Type'  => 'application/json',
        ],
        'body' => json_encode([
            'model' => 'gpt-3.5-turbo',
            'messages' => [
                ['role' => 'user', 'content' => $prompt]
            ],
            'max_tokens' => 100,
            'temperature' => 0.7
        ])
    ]);
    if (is_wp_error($response)) {
        echo 'Error: ' . $response->get_error_message();
        wp_die();
    }

    $body = json_decode(wp_remote_retrieve_body($response), true);
    $output = $body['choices'][0]['message']['content'] ?? 'No response from AI.';
    echo nl2br(esc_html($output));

    wp_die();
}
?>
