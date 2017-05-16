---
title: Optimizing list render performance in React Native
description: Rendering large lists can quickly become very slow. Insert FlatList.
layout: post
image: /assets/images/flat-list-demo.jpg
---

Coming from a primarily web front end and backend developer, I have learned to take some things for granted. On todays modern hardware, I don't spend a ton of time thinking about how something might result in slow render performance, since that usually isn't an issue. But now that I've been doing a lot of mobile development with React Native, some patterns have come around to bite me primarily because of the reduced processing power on the mobile device.

Lists and grid views are a common pattern in mobile apps. Several co workers and I were tasked with building out part of our app that would consist of many scrollable views that were all fairly complex. In the outermost scrolling view, a single item could be rendering upwards of 500 views by itself. Each item in the list had a large amount of content hidden in a drawer that could be pulled out from the bottom of the view using a swipe gesture. It was within this drawer that we had another srollable view that was the primary culprit in needing to render a large amount of views. When I wrote the first iteration of this scrollable view I used a naive approach using [`ScrollView`](https://facebook.github.io/react-native/docs/scrollview.html) and later switched to using [`FlatList`](https://facebook.github.io/react-native/docs/flatlist.html) which took inital render times from over 7 seconds to around 1 second.

React has a ton of optimizations for making updates to the render tree extremely performant, such as virtual dom diffing and the `shouldComponentUpdate` method on a component. But when you are doing an initial render, the only such optimization, is to just not render as much. That can be hard though when rendering a long list of complex views.

Here's an example of how switching from `ScrollView` to `FlatList` can drastically reduce initial render times.
In this example app we are building a list containing 100 rows that render 100 `Text` views with the numbers 0-99

![Demo app with rows with the numbers 0-99](/assets/images/flat-list-demo.jpg)

In the first iteration we are going to render this using a scroll view. The entry point component looks like

{% highlight javascript %}
export default class FlatListPerformance extends Component {
  // create 100 rows
  prepData = () => {
    let data = []
    for (let i = 0; i < 100; i++) {
      data.push({ key: i })
    }
    return data
  }

  // render each row by creating 100 Text components in each row 
  renderItem = ({ item }) => {
    let texts = []
    for (let i = 0; i < 100; i++) {
      texts.push(<Text key={i} style={styles.text}>{i}</Text>)
    }
    return (
      <View style={styles.item} key={item.key}>
        {texts}
      </View>
    )
  }

  // render a scroll view with each of the items
  render() {
    return (
      <ScrollView style={styles.container}>
        {this.prepData().map((item) => this.renderItem({ item }))}
      </ScrollView>
    );
  }
}
{% endhighlight %}

This naive approach is very slow. Taking upwards of 12 seconds to render from when the app opens to when you actually see content in the UI.

![Animated GIF of the demo app rendering the scroll view](/assets/images/scroll-view.gif)

If we capture a CPU profile of this render process, we see that the JS runtime is spending around 2 seconds to determine what the render tree looks like. 

![A cpu profile flame chart](/assets/images/scroll-view-cpu-profile.jpg)

After the green section, the JS runtime goes idle while it waits for the native side to render all of the views that were generated, taking around 10 seconds. As a side note, I added a 2 second render delay when the app first starts so I had time to start the cpu profiler. So the first idle time is during that waiting period.

`FlatList` can immediately improve native side rendering. It's a common pattern in iOS development to have lists recycle views. So if you have a very long list, the app is actually only rendering the views that are actually on screen. As you scroll through the list, and one of the rows leaves the screen, it will get recycled and added to the other side of the list with the data for the next row in the list. This improves memory usage for an app drastically as it is not required to allocate memory for every row in a list. But it has another advantage of only rendering views that are in screen. Which means that views that are not on screen won't even get rendered. `FlatList` behaves much like these native UI list views. `FlatList` exposes this behavior to lists in React Native which can drastically decrease render time for complex lists.

If we change the render method of our entry component to use `FlatList` we can see the immediate render benefits.

{% highlight javascript %}
render() {
  return (
    <FlatList
      data={this.prepData()}
      renderItem={this.renderItem}
      styles={styles.container}
    />
  );
}
{% endhighlight %}

![Demo app being rendered using FlatList](/assets/images/flat-list.gif)

While this improves things drastically, `FlatList` allows us to take it one step further. While `FlatList` improves render performance on the native side, it seems to still render the whole tree on the JS side. `FlatList` has a prop that you can pass to it named [`initialNumToRender`](https://facebook.github.io/react-native/docs/flatlist.html#initialnumtorender) which will tell it to only render a certain number of rows on the JS side. Then after it renders, to immediately rerender with all of the rows. When using this prop it is advised to set it to the number of rows that fill the screen so that rendering does not look jarring to the end user. In the case of this demo app 4 rows will fill the view. If the rows in the list are complex at all it also helps to ensure your row component either extends `PureComponent` or implements `shouldComponentUpdate`

{% highlight javascript %}
render() {
  return (
    <FlatList
      data={this.prepData()}
      renderItem={this.renderItem}
      styles={styles.container}
      initialNumToRender={4}
    />
  );
}
{% endhighlight %}

![Demo app being rendered with initialNumToRender](/assets/images/initial-num-to-render.gif)

Just like that we take a scrollable view that was taking 12+ seconds to render and get it to render in less than a second. Another thing that will improve render performance is running the app in Release mode. This will tell React to not check proptypes on render. All react native components have prop types defined on them which can eat a lot of time in rendering.

This chart is from the cpu profile in the image above. We can see that `checkPropTypes` is taking almost 300ms, which was more than 10% of the time we spent in the JS generating the render tree.

![A cpu function chart](/assets/images/profile-chart.jpg)
