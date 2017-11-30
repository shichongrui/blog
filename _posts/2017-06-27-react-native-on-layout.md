---
title: onLayout in React Native
description: How to get the dimensions of a view in React Native
image: /assets/images/react-native-on-layout.jpg
layout: post
---

#### Update 11/28/2017
Some readers (Kurt and Michael thanks!) pointed out an error I had in one of my examples and on top of that the `react-native-on-layout` component is no longer an HOC but a render prop component now so I thought I would give this post an update.

A common pattern in React Native is changing layout/sizes of views based on the size of its parent view. We can use React Native's implementation of flex box to get us 90% of the way there. But there are just some layouts that flex box just has a hard time with. In the app I am building at work, there are two common instances where this is the case. When an absolutely positioned view needs to adjust its size and/or position based on its parent, as well as grid views where each item in the grid has a flexible width and height based on the size of the parent.

There are two ways to get the size of a rendered component in React Native. One is a passive method while the other is an active method. The active method is to use react refs and a method called `measure`. This method as it turns out doesn't work well for these two problems as we would need to know every time the view changes and ask for the dimensions again. Unfortunately we may not know when the view changes and so React Native give us a different way to passively obtain the view size which is `onLayout`.

`onLayout` is a pretty simple method. It is called anytime the React Native runtime performs a layout of your component. It is an event handler that you pass in as a prop to any React Native built in componenent that originates from a View. Your method will get called with an event object which will look like `event.nativeEvent.layout.width/height/x/y` The `x` and `y` values are relative to the top left corner of the screen. Lets say we are creating an avatar component:

{% highlight javascript %}
class Avatar extends Component {
  render (
    <View>
      <Image source={% raw %}{{ uri: 'http://fillmurray.com/200/200' }}{% endraw %} width={200} height={200} />
      <Text>My Picture</Text>
    </View>
  )
}
{% endhighlight %}

Using `onLayout` if we wanted to know the rendered size of our component we would provide the `onLayout` prop to the `View` component.

{% highlight javascript %}
class Avatar extends Component {
  onLayout = (e) => {
    this.setState({
      width: e.nativeEvent.layout.width,
      height: e.nativeEvent.layout.height,
      x: e.nativeEvent.layout.x,
      y: e.nativeEvent.layout.y
    })
  }

  render (
    <View onLayout={this.onLayout}>
      <Image source={% raw %}{{ uri: 'http://fillmurray.com/200/200' }}{% endraw %} width={200} height={200} />
      <Text>My Picture</Text>
    </View>
  )
}
{% endhighlight %}

Now we can use `this.state.width` and `this.state.height` to do any sort of calculations we need to in our component using the rendered width and height of our component.

I’ve had to do this enough times that I decided to make a component that makes doing this a little less cumbersome.

`react-native-on-layout` is an npm package that you can include in your app using `npm i react-native-on-layout` or with `yarn add react-native-on-layout`. It is a `View` component that will do this `onLayout` dance for you and then passes the layout values as parameters to a render prop. Lets take the example of our Avatar. We can make our component a stateless function component and then use `react-native-on-layout` to get the width and height.

{% highlight javascript %}
import onLayout from 'react-native-on-layout'

const Avatar = () => (
  <OnLayout>
    {({ width, height, x, y }) => (
      <Image source={% raw %}{{ uri: 'http://fillmurray.com/200/200' }}{% endraw %} width={200} height={200} />
      <Text>
        My Picture {width} {height} {x} {y}
      </Text>  
    )}
  </OnLayout>
)

// use in a render method
<Avatar />
{% endhighlight %}


