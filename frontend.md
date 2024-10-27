# Performance guidlines for frontend

- https://www.abondance.com/actualites/20180615-19309-infographies-35-actions-pour-accelerer-son-site.html

Assets :
- Minify, concat and compress assets
- Use CDN
- Optimize images : https://developers.google.com/speed/docs/insights/OptimizeImages

HTML :
- Adaptive images
- Image lazy loading
- Content lazy loading
- Preload, prefetch, preconnect
- W3C valid
- Use `async` and `defer`
- Clean up HTML

JavaScript/CSS :
- Mobile first
- Avoid external libraries (jQuery, Bootstrap, Moment, Lodash)
- Avoid animations
- Avoid render-blocking code
- Use web-safe fonts (https://www.w3schools.com/cssref/css_websafe_fonts.asp)
- Compatibility with old browsers
- Avoid `@import`
- Batch DOM changes

Features :
- Remove non-essential features
- Use static pages

Metrics to monitor :
- LCP
- FID
- CLS

Read more:
- https://www.youtube.com/playlist?list=PLNYkxOF6rcICVl6Vb-AFlw81bQLuv6a_P
