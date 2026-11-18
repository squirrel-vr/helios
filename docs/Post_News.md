# News Posting Instructions

1. Open file `rss.xml`
2. locate tag `<channel>`
3. Locate comment `<!-- items go here -->`
4. create a new item on the top of existing ones (it should be the first item). Use this template (I put fields that need to be edited in curly brackets but in the file you don't need them):
    ```xml
    <item>
        <title>{Article title}</title>
        <pubDate>{Sun, 1 Nov 2026 18:00:00 +0300}</pubDate>
        <dc:creator>{Author (e.g. you)}</dc:creator>
        <content:encoded>
            <![CDATA[
                {html content of the article}
            ]]>
        </content:encoded>
    </item>
    ```
5. Fill in the html content. Supported tags are: `<p>`, `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`, `<ul>`, `<ol>`, `<li>`, `<a>`, `<img>`, `<br/>` (use previous articles as a reference)
6. Save the file, commit, and push the changes to the repository: 
    ```
    git add .
    git commit -m "Add new news item"
    git push
    ```