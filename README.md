# picolin64.github.io

Personal cybersecurity blog.

This site is based on the [Zooper](https://github.com/fayazara/zooper) template. Credits to [Fayaz Ahmed](https://github.com/fayazara).

## Tech Stack

1. Nuxt JS
2. Tailwind CSS
3. Vue
4. Nuxt Content Module
5. Shiki JS ES

## Installation

1. ``git clone`` this repo.
2. ``cd`` into the project directory.
3. Run ``npm install``.
4. Run ``npm run dev`` to start a local development server.

## Deploy changes

1. Run ``npm run deploy``. This command builds the application, pre-renders every route as a HTML file (GitHub Pages only supports static hosting), creates the .nojekyll file to avoid conflicts with resource loading and pushes the content generated to the "gh-pages" branch (you don't need to create this branch before running the command).

    **Important:** If you get any of the following errors after executing the last command:

    ```text
    FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory

    FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
    ```

    Then you need to increase the default amount of memory allocated by Node.js to a minimum of 8GB. See <https://bobbyhadz.com/blog/javascript-heap-out-of-memory#setting-the-node_options-environment-variable-on-windows> for further instructions.

2. Commit and push changes to your GitHub repo.
3. In the GitHub repo, go to the Settings tab, then Pages menu and select "gh-pages" as the branch the site will be built. Save changes.
4. Wait for deploy.

You need to follow steps 1 and 2 every time you make any changes. Step 3 only needs to be done once for the first deployment.

## To do

- Implement English language.
- Add navigation menu for page sections.
- Add navigation menu for writeups grouped according to their respective platform.
- Add tags to pages.
- Implement a search bar.
