+++
title = 'How to parallelize Webpack Loaders'
date = 2024-05-18T16:35:24-03:00
draft = false
+++

Runing webpack on a big project can be an agony for developers and can delay the production builds and CI if is not treated properly.

My aim here is to present how you can parallelize the webpack loaders and reduce the times to just seconds.

## Motivation
Working with a very big project built on angularjs (a monolithic with 5+ products inside) and with a custom webpack configuration that managed a lot of different file types, the building time of the app was around 7' only for webpack tasks.
So i decided to parallelize the most heavy loaders (like typescript and html) and reduce the struggle of the team and the cost of the builds.

## Tools
For the speed check of webpack work, i used, and highly recomend [SMP](https://github.com/stephencookdev/speed-measure-webpack-plugin). With this package you only need to wrap your webpack object and it messures the times it takes for each loader to finish.
For the parallelization i used [HappyPack](https://github.com/amireh/happypack). Which allows to set certain number of threads for every loader to work.
So each loader can transcript the 

## Original webpack config
The following configuration for webpack rules worked ok, did the job, but with such a big amount of files it took so long to finish.
Here we have js, ts, css, html, scss, icons and image types to be packaged by webpack (js and ts are separated because of the necessity of using the plugin "angularjs-annotate" for production):

```javascript
module.exports = smp.wrap({
    [...],
    module: {
        rules: [{
            test: /\.ts?$/,
            use: [
                {
                    loader: 'babel-loader?cacheDirectory',
                    options: {  
                        plugins: ["angularjs-annotate", "syntax-dynamic-import"] 
                    }
                },
                { loader: 'ts-loader' } 
            ]
        },
        {
            test: /\.html?$/,
            loader: 'html-loader'
        },
        {
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
        },
        { 
            test: /\.scss$/,
            use: [{ loader: "style-loader"  }, { loader: "css-loader" }, { loader: "sass-loader" }]
        },
        { 
            test: /\.(woff|woff2|eot|ttf)$/,
            loader: 'url-loader?limit=100000'
        },
        { 
            test: /\.(png|jpg|gif|svg|json|xml|ico)$/,
            loader: 'file-loader'
        },
        { 
            test: /\.js$/, 
            exclude: /(node_modules)/,
            use: {
                loader: 'babel-loader?cacheDirectory',
                options: { 
                    presets: [['es2015', {
                        "targets": "> 0.25%",
                        "useBuiltIns": "usage"
                    }]],  
                } 
            }
        }]
    }
})
```

Measured with smp it showed that the most expensive loader was ts-loader (almost 7 minutes), having to manage a lot of files from the project. So is the first to work with.

## Parallelizing

So incorporating HappyPack to webpack is very straight forward, first you have to include a HappyPack instance with the plugins:

```javascript
plugins: [
    new HappyPack({
        id: 'ts',
        threads: 2,
        loaders: [
            {
                path: 'ts-loader',
                query: { happyPackMode: true }
            }
        ]
    }),
    new HappyPack({
        id: 'babel-ts',
        threads: 2,
        loaders: [{
            path: 'babel-loader',
            happyPackMode: true,
            query: {
                plugins: ["angularjs-annotate", "syntax-dynamic-import"],
                cacheDirectory: true
            }
        }]
    })
]
```

Here we have an *id* property that will bind the plugin instance with the correspondent rule, the number of threads wanted, and the loaders to be parallelized. I only used 2 threads for each instance because testing it it didn't change the performance when i incremented the number of threads.
Here i have 2 instances of HappyPack because i wanted to parallelize both loaders in my *ts* rule, one for ts-loader, and one for babel-loader (needed for the angularjs-annotate and dynamic imports in production).
After we define our plugin called 'ts', we have to modify the rule:

```javascript
{
	test: /\.ts?$/,
	use: [
		{ loader: 'happypack/loader?id=babel-ts' },
		{ loader: 'happypack/loader?id=ts' }
	]
},
```

So now, when webpack wants to work with the rule correspondent to the *.ts* file type, it will be looking for an HappyPack instance with the passed id, and run it according to its configuration.

## Results
Parallelizing the `ts-loader`, the time it took to run accordingly to SMP changed from 7 minutes, to only 30 seconds. Which is a very significant proportion.
The costs of building the app reduced a lot, and (most important) the development team was very pleased with the less time waiting for each run.
This same work with the `ts-loader` was replicated also in other loaders, as  the html-loader.
With loaders that doesn't parse too many files it doesn't make such difference and can be a waste of time, so the rest remains on a single thread.
Also it has to be said HappyPack doesn't work with all webpack loaders as some loader API are not supported.

## Conclusion
Using this very simple and very straight forward tool you can reduce significatly the times of you webpack tasks and benefit the budgets and the dev team.