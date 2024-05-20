+++
title = 'How to use Hotjar JavaScript triggers on Angular'
date = 2024-05-19T16:42:23-03:00
draft = false
+++

JavaScript triggers are the most useful feature for using Hotjar HeatMaps in one page applications, because without that you can't create HeatMaps on dynamic content or modals, dropdowns, etc.
So i will show you how i made it work on an Angular project.
I'm making the assumption that you already have an Hotjar account and initialize a project.

## Include Hotjar in your project

First we need to include the tracking code in our `index.html`

```html
<!-- Hotjar Tracking Code for https://yourpage.com/ -->
<script>
   (function(h,o,t,j,a,r){
       h.hj = h.hj || function(){ ( h.hj.q = h.hj.q || []).push(arguments) };
       h._hjSettings = { hjid:xxxxxxx, hjsv:x };
       a = o.getElementsByTagName('head')[0];
       r = o.createElement('script'); r.async = 1;
       r.src = t + h._hjSettings.hjid + j + h._hjSettings.hjsv;
       a.appendChild(r);
   })(window,document,'https://static.hotjar.com/c/hotjar-','.js?sv=');
</script>
```

With that we already have the hj function on our window object.

## Hotjar Service

Now my goal was to make the access to the hj function reliable and mantainable across the Angular aplication, so i made a service:

```typescript
@Injectable()
export class HotjarService {
  public getInstance() {
    return window['hj'] = window['hj'] || this.getHotjarQueue(); 
  }

  public trigger(eventName: string) {
    try {
      this.getInstance()('trigger', eventName);
    } catch (err) {
      console.error(err);
    }
  }

  public vpv(url: string) {
    try {
      this.getInstance()('vpv', url);
    } catch (err) {
      console.error(err);
    }
  }

  private getHotjarQueue() {
    return function () {
      (window['hj']['q'] = window['hj']['q'] || []).push(arguments);
    };
  }
}
```

Here we have the `trigger` function that allows us to register a new HeatMap, and the `vpv` function that register a new virtual page. The `getInstance` and the `getHotjarQueue` expose the `window.hj` if available to the service.

## Conclusion
Making this little wrapper to the `windows.hj` we can make the use of Hotjar a lot more readable and mantainable in a big project. Using the function directly from the `window` object in frameworks as Angular is dangerous because of side effects and untraceable errors.