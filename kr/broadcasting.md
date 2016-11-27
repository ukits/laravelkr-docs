# Event Broadcasting
# 이벤트 브로드캐스팅

- [Introduction](#introduction)
- [소개](#introduction)
    - [Configuration](#configuration)
    - [설정](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
    - [드라이버 전제조건](#driver-prerequisites)
- [Concept Overview](#concept-overview)
- [컨셉 개요](#concept-overview)
    - [Using Example Application](#using-example-application)
    - [예제 애플리케이션 사용하기](#using-example-application)
- [Defining Broadcast Events](#defining-broadcast-events)
- [브로드캐스트 이벤트 정의하기](#defining-broadcast-events)
    - [Broadcast Data](#broadcast-data)
    - [브로드캐스트 데이터](#broadcast-data)
    - [Broadcast Queue](#broadcast-queue)
    - [브로드캐스트 큐](#broadcast-queue)
- [Authorizing Channels](#authorizing-channels)
- [승인 채널](#authorizing-channels)
    - [Defining Authorization Routes](#defining-authorization-routes)
    - [승인 라우트 정의하기](#defining-authorization-routes)
    - [Defining Authorization Callbacks](#defining-authorization-callbacks)
    - [승인 콜백 정의하기](#defining-authorization-callbacks)
- [Broadcasting Events](#broadcasting-events)
- [브로드캐스팅 이벤트](#broadcasting-events)
    - [Only To Others](#only-to-others)
    - [오직 다른이에게만](#only-to-others)
- [Receiving Broadcasts](#receiving-broadcasts)
- [브로드캐스트 수신하기](#receiving-broadcasts)
    - [Installing Laravel Echo](#installing-laravel-echo)
    - [라라벨 에코 설치하기](#installing-laravel-echo)
    - [Listening For Events](#listening-for-events)
    - [이벤트 리스닝](#listening-for-events)
    - [Namespaces](#namespaces)
    - [네임스페이스](#namespaces)
- [Presence Channels](#presence-channels)
- [주둔 채널](#presence-channels)
    - [Authorizing Presence Channels](#authorizing-presence-channels)
    - [주둔 채널 승인하기](#authorizing-presence-channels)
    - [Joining Presence Channels](#joining-presence-channels)
    - [주둔 채널에 들어가기](#joining-presence-channels)
    - [Broadcasting To Presence Channels](#broadcasting-to-presence-channels)
    - [주둔 채널에 브로드캐스트하기](#broadcasting-to-presence-channels)
- [Notifications](#notifications)
- [알림](#notifications)

<a name="introduction"></a>
## Introduction
## 소개

In many modern web applications, WebSockets are used to implement realtime, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a WebSocket connection to be handled by the client. This provides a more robust, efficient alternative to continually polling your application for changes.
많은 현대적인 웹 어플리케이션들은 웹 소캣을 이용하여 실시간 어플리케이션을 구현하며 UI를 실시간으로 업데이트합니다. 일반적으로 서버에 데이터가 변경되면 메세지가 웹 소캣 연결로 보내져 클라이언트에 의해 처리될 수 있도록 합니다. 웹 소켓은 당신의 애플리케이션의 변화를 지속적으로 폴링하는 더 강력하고 효과적인 대안을 제공합니다.

To assist you in building these types of applications, Laravel makes it easy to "broadcast" your [events](/docs/{{version}}/events) over a WebSocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript application.
이러한 류의 애플리케이션을 만드는걸 돕기 위해, 라라벨은 당신의 이벤트를 웹소켓 너머로 "브로드캐스트"하기 쉽게 만들었습니다. 라라벨 이벤트를 브로드캐스트하면 여러분의 서버 쪽 코드와 클라이언트 쪽 자바스크립트 프레임워크가 같은 이벤트 이름을 공유할 수 있습니다.

> {tip} Before diving into event broadcasting, make sure you have read all of the documentation regarding Laravel [events and listeners](/docs/{{version}}/events).
> {tip} 브로드캐스팅에 대해 더 알아보기 전에, 라라벨 [이벤트와 리스너](/docs/{{version}}/events)를 읽으세요.

<a name="configuration"></a>
### Configuration
### 설정

All of your application's event broadcasting configuration is stored in the `config/broadcasting.php` configuration file. Laravel supports several broadcast drivers out of the box: [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), and a `log` driver for local development and debugging. Additionally, a `null` driver is included which allows you to totally disable broadcasting. A configuration example is included for each of these drivers in the `config/broadcasting.php` configuration file.
모든 이벤트 브로드캐스팅 설정 옵션은 config/broadcasting.php 의 설정 파일에 저장되어 있습니다. 라라벨은 기본적으로 여러가지의 브로드캐스트 드라이버([Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), 그리고 디버깅 용도의 `log` 드라이버)를 지원합니다. 추가적으로 전체적으로 브로드캐스팅을 끌 수 있도록 해주는 `null` 드라이버도 제공됩니다. 각각의 드라이버의 설정 예제는 `config/broadcasting.php` 설정 파일에 있습니다.

#### Broadcast Service Provider
#### 브로드캐스트 서비스 프로바이더

Before broadcasting any events, you will first need to register the `App\Providers\BroadcastServiceProvider`. In fresh Laravel applications, you only need to uncomment this provider in the `providers` array of your `config/app.php` configuration file. This provider will allow you to register the broadcast authorization routes and callbacks.
어떤 이벤트던 브로드캐스팅하려면 그 전에, `App\Providers\BroadcastServiceProvider`를 등록해야 합니다. 최신 라라벨 애플리케이션에서는, `config/app.php` 설정 파일에 있는 `providers` 배열에서 이 프로바이더의 주석을 해제 해주기만 하면 됩니다. 이 프로바이더는 브로드캐스트 인증 라우트와 콜백을 등록할 수 있게 해줍니다.

#### CSRF Token
#### CSRF 토큰

[Laravel Echo](#installing-laravel-echo) will need access to the current session's CSRF token. If available, Echo will pull the token from the `Laravel.csrfToken` JavaScript object. This object is defined in the `resources/views/layouts/app.blade.php` layout that is created if you run the `make:auth` Artisan command. If you are not using this layout, you may define a `meta` tag in your application's `head` HTML element:
현재 세션의 CSRF 토큰에 접근하기 위해 [Laravel Echo](#installing-laravel-echo)가 필요할 것입니다. 만약 가능하다면, Echo는 `Laravel.csrfToken` 자바스크립트 객체로부터 토큰을 받아 올 것입니다. 이 객체는 `make:auth`라는 아티즌 커맨드를 실행하면 생성되는 `resources/views/layouts/app.blade.php` 레이아웃에 정의되어 있습니다. 만약 당신이 이 레이아웃을 사용하지 않는다면, HTML `head`에 `meta` 태그를 정의할 수 있습니다.

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Driver Prerequisites
### 드라이버 전제조건들

#### Pusher

If you are broadcasting your events over [Pusher](https://pusher.com), you should install the Pusher PHP SDK using the Composer package manager:
만약 [Pusher](https://pusher.com) 를 이용해서 이벤트를 브로드캐스팅한다면, 컴포저 패키지 매니저를 사용해 Pusher PHP SDK를 설치해야 합니다.

    composer require pusher/pusher-php-server

Next, you should configure your Pusher credentials in the `config/broadcasting.php` configuration file. An example Pusher configuration is already included in this file, allowing you to quickly specify your Pusher key, secret, and application ID.
그리고 나서, `config/broadcasting.php` 설정 파일에서 Puhser 암호를 설정해야 합니다. Pusher key, 비밀번호, 그리고 애플리케이션 ID를  빠르게 설정할 수 있도록 Puhser 설정 예제가 이미 이 파일에 포함되어 있습니다.

When using Pusher and [Laravel Echo](#installing-laravel-echo), you should specify `pusher` as your desired broadcaster when instantiating an Echo instance:
Pusher 와 [Laravel Echo](#installing-laravel-echo) 를 쓸 때는, Echo 인스턴스를 초기화할 때 `pusher`를 브로드캐스터로 지정해주어야 합니다.

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis
#### Redis

If you are using the Redis broadcaster, you should install the Predis library:
만약 Redis 브로드캐스터를 사용한다면, Predis 라이브러리를 설치해야 합니다.

    composer require predis/predis

The Redis broadcaster will broadcast messages using Redis' pub / sub feature; however, you will need to pair this with a WebSocket server that can receive the messages from Redis and broadcast them to your WebSocket channels.
Redis 브로드캐스터는 레디스의 pub / sub 기능을 이용해 메시지를 브로드캐스트 할 것입니다. 하지만 Redis로 부터 메시지를 받아 당신의 웹소캣 채널들에 브로드캐스트 할 수 있도록 레디스와 웹소캣 서버를 페어링해야 합니다.

When the Redis broadcaster publishes an event, it will be published on the event's specified channel names and the payload will be a JSON encoded string containing the event name, a `data` payload, and the user that generated the event's socket ID (if applicable).
Redis 브로드캐스터가 이벤트를 발행하면, 이벤트는 이벤트에 명시된 채널명에 발행될 것이고 payload는 이벤트명, `data` payload, 그리고 이벤트 소켓 ID를 생성한 사용자를 담고 있는 JSON 인코딩된 문자가 될 것입니다.

#### Socket.IO
#### Socket.IO

If you are going to pair the Redis broadcaster with a Socket.IO server, you will need to include the Socket.IO JavaScript client library in your application's `head` HTML element:
만약 Redis 브로드캐스터와 Socket.IO 서버를 페어링 하고자 한다면, 애플리케이션의 `head`에 Socket.IO 자바스크립트 클라이언트 라이브리를 인클루드 해야 합니다.

    <script src="https://cdn.socket.io/socket.io-1.4.5.js"></script>

Next, you will need to instantiate Echo with the `socket.io` connector and a `host`. For example, if your application and socket server are running on the `app.dev` domain you should instantiate Echo like so:
그리고나서, `socket.io` 커넥터와 `host`로 Echo를 초기화 시켜야 합니다. 예를 들어, 당신의 애플리케이션과 소켓 서버가 `app.dev` 도메인에서 운영되고 있다면 당신은 Echo를 다음과 같이 초기화 해야 합니다.

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: 'http://app.dev:6001'
    });

Finally, you will need to run a compatible Socket.IO server. Laravel does not include a Socket.IO server implementation; however, a community driven Socket.IO server is currently maintained at the [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub repository.
마지막으로, 호환되는 Socket.IO 서버를 실행할 필요가 있습니다. 라라벨은 Socket.IO 서버 구현을 포함하지 않고 있습니다. 하지만 커뮤니티가 주도하는 Socket.IO 서버가 현재 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) 깃헙 라이브러리에서 관리되고 있습니다.

#### Queue Prerequisites
#### 큐 전제조건

Before broadcasting events, you will also need to configure and run a [queue listener](/docs/{{version}}/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected.
이벤트를 브로드캐스트 하기전에, [큐 리스너](/docs/{{version}}/queues)도 설정하고 실행해야 합니다. 모든 이벤트 브로드캐스팅은 대기 행렬화된 작업(queued jobs)을 통해 처리되기 때문에 애플리케이션의 반응 시간에 크게 영향을 미치지 않습니다.

<a name="concept-overview"></a>
## Concept Overview
## 컨셉 개요

Laravel's event broadcasting allows you to broadcast your server-side Laravel events to your client-side JavaScript application using a driver-based approach to WebSockets. Currently, Laravel ships with [Pusher](http://pusher.com) and Redis drivers. The events may be easily consumed on the client-side using the [Laravel Echo](#installing-laravel-echo) Javascript package.
라라벨의 이벤트 브로드캐스팅은 웹소캣에 드라이버 기반 접근법을 사용하여 서버측의 라라벨 이벤트를 클라이언트측의 자바스크립트 애플리케이션에 전송할 수 있도록 해줍니다. 현재 라라벨은 [Pusher](http://pusher.com)와 Redis 드라이버를 제공합니다. 이벤트는 클라이언트단에서 [Laravel Echo](#installing-laravel-echo)라는 자바스크립트 패키지를 통해 손쉽게 사용할 수 있습니다.

Events are broadcast over "channels", which may be specified as public or private. Any visitor to your application may subscribe to a public channel without any authentication or authorization; however, in order to subscribe to a private channel, a user must be authenticated and authorized to listen on that channel.
이벤트는 공개적이거나 비공개적이라고 명시된 "채널"을 통해 브로드캐스트 됩니다. 모든 방문자는 인증이나 승인 없이도 공개 채널을 구독할 수 있습니다. 하지만 비공개 채널을 구독하기 위해서는 반드시 인증과 승인을 받아야 합니다.

<a name="using-example-application"></a>
### Using Example Application
### 예제 애플리케이션 사용하기

Before diving into each component of event broadcasting, let's take a high level overview using an e-commerce store as an example. We won't discuss the details of configuring [Pusher](http://pusher.com) or [Laravel Echo](#echo) since that will be discussed in detail in other sections of this documentation.
이벤트 브로드캐스팅의 각 구성요소에 대해 깊게 들어가기 전에, 전자상거래 상점을 예로들어 전반적인 내용을 둘러보도록 하겠습니다. 이 문서의 다른 부분에서 따로 자세하게 다룰 것이기 때문에 [Pusher](http://pusher.com) 나 [Laravel Echo](#echo) 설정에 대한 자세한 내용은 다루지 않습니다.

In our application, let's assume we have a page that allows users to view the shipping status for their orders. Let's also assume that a `ShippingStatusUpdated` event is fired when a shipping status update is processed by the application:
애플리케이션에 사용자들이 자신의 주문에 대해 배송상태를 확인할 수 있는 페이지가 있다고 생각해봅시다. 그리고 애플리케이션에 의해 배송 상태가 업데이트 되면 `ShippingStatusUpdated` 라는 이벤트가 발생한다고 가정해봅시다.

    event(new ShippingStatusUpdated($update));

#### The `ShouldBroadcast` Interface
#### `ShouldBroadcast` 인터페이스

When a user is viewing one of their orders, we don't want them to have to refresh the page to view status updates. Instead, we want to broadcast the updates to the application as they are created. So, we need to mark the `ShippingStatusUpdated` event with the `ShouldBroadcast` interface. This will instruct Laravel to broadcast the event when it is fired:
우리는 사용자가 그들의 주문을 조회하고 있을 때, 배송 상태가 업데이트 된 것을 보기 위해 페이지를 반드시 리프레시 해야하는 것을 원하지 않습니다. 대신, 갱신되었음을 애플리케이션에 전송하길 원합니다. 우리는 `ShouldBroadcast` 인터페이스로 `ShippingStatusUpdated` 이벤트를 표시할 필요가 있습니다. 이렇게하면 라라벨에게 이벤트가 발생시 이 이벤트를 전송하라고 지시할 것입니다.

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        //
    }

The `ShouldBroadcast` interface requires our event to define a `broadcastOn` method. This method is responsible for returning the channels that the event should broadcast on. An empty stub of this method is already defined on generated event classes, so we only need to fill in its details. We only want the creator of the order to be able to view status updates, so we will broadcast the event on a private channel that is tied to the order:
`ShouldBroadcast` 인터페이스는 `broadcastOn` 메소드를 정의하기 위해서 이벤트를 필요로 합니다. 이 메소드는 이벤트가 전송되어야 할 채널을 돌려주는 역할을 합니다. 생성된 이벤트 클래스에 비어있는 메소드가 이미 정의되어 있기 때문에, 구체적인 내용만 채워넣으면 됩니다. 우리는 주문을 생성한 사람들만이 갱신된 상태를 볼 수 있도록 하고 싶으므로, 이 주문과 관련된 채널들에만 비공개적으로 이벤트를 전송할 것입니다.  

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Authorizing Channels
#### 채널 인증하기

Remember, users must be authorized to listen on private channels. We may define our channel authorization rules in the `boot` method of the `BroadcastServiceProvider`. In this example, we need to verify that any user attempting to listen on the private `order.1` channel is actually the creator of the order:
비공개 채널을 수신하기 위해서는 반드시 사용자가 인증되어야 합니다. 우리는 `BroadcastServiceProvider` 의 `boot` 메소드에 채널 인증 규칙을 정의할 수 있습니다. 이 예제에서는 비공개 채널인 `order.1` 채널을 수신하고자 시도하는 모든 유저에 대해 해당 주문을 실제로 생성한 사람인지 확인할 필요가 있습니다.

    Broadcast::channel('order.*', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.
`channel` 메소드는 두 개의 인자(채널명과 사용자가 채널을 들을 수 있도록 인증되었는지를 나타내는 `true` 혹은 `false`를 되돌려주는 콜백)를 받습니다.  

All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `*` character to indicate that the "ID" portion of the channel name is a wildcard.
모든 승인 콜백은 현재 인증된 사용자를 첫번째 인수로 받고, 다음 인수로 추가적인 와일드카드 파라미터들을 받습니다. 이 예제에서, 채널명의 "ID" 부분을 가리키기 위해 `*`를 사용하는 것이 와일드 카드입니다.

#### Listening For Event Broadcasts
#### 이벤트 브로드캐스트 수신하기

Next, all that remains is to listen for the event in our JavaScript application. We can do this using Laravel Echo. First, we'll use the `private` method to subscribe to the private channel. Then, we may use the `listen` method to listen for the `ShippingStatusUpdated` event. By default, all of the event's public properties will be included on the broadcast event:
다음으로 자바스크립트 애플리케이션에서 이벤트를 수신하는 것만 남았습니다. 라라벨 에코를 사용해서 이를 처리할 수 있습니다. 우선, 비공개 채널을 수신하기 위해 `private` 메소드를 사용할 것입니다. 그리고 나서, `ShippingStatusUpdated` 이벤트를 수신하기 위해 `listen` 메소드를 사용할 것입니다. 기본적으로, 이벤트의 모든 공개 속성들은 전송하는 브로드캐스트 이벤트에 포함됩니다.

    Echo.private('order.' + orderId)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Defining Broadcast Events
## 브로드캐스트 이벤트 정의하기

To inform Laravel that a given event should be broadcast, implement the `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class. This interface is already imported into all event classes generated by the framework so you may easily add it to any of your events.
라라벨에게 어떤 이벤트가 브로드캐스트 되어야 한다고 알려주기 위해, 이벤트 클래스에 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 인터페이스를 구현합니다. 이 인터페이스는 프레임워크에 의해 생성된 모든 이벤트 클래스에 이미 포함되어 있으므로, 어떤 이벤트던 이를 쉽게 추가할 수 있을 것입니다.

The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`. The `broadcastOn` method should return a channel or array of channels that the event should broadcast on. The channels should be instances of `Channel`, `PrivateChannel`, or `PresenceChannel`. Instances of `Channel` represent public channels that any user may subscribe to, while `PrivateChannels` and `PresenceChannels` represent private channels that require [channel authorization](#authorizing-channels):
`ShouldBroadcast` 인터페이스는 `broadcastOn`이라는 단 하나의 메소드를 구현할 것을 요구합니다. `broadcastOn` 메소드는 이벤트를 브로드캐스트 해야하는 채널 혹은 채널의 배열을 반환해야 합니다. 채널은 `Channel`, `PrivateChannel` 혹은 `PresenceChannel` 의 인스턴스여야 합니다. `Channel`의 인스턴스는 모든 사용자가 구독할 수 있는 공개 채널을 나타내는 반면, `PrivateChannels`와 `PresenceChannels`는 [채널 승인(channel suthorization)](#authorizing-channels)을 필요로 하는 비공개 채널을 나타냅니다.

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Then, you only need to [fire the event](/docs/{{version}}/events) as you normally would. Once the event has been fired, a [queued job](/docs/{{version}}/queues) will automatically broadcast the event over your specified broadcast driver.
그러고나면, 당신이 평소에 하듯 이벤트를 발생시키기만 하면 됩니다. 이벤트가 한 번 발생하고나면, [대기행렬화된 작업(queued job)](/docs/{{version}}/queues)이 자동으로 당신이 지정한 브로드캐스트 드라이버를 통해 이벤트를 자동으로 브로드캐스트 할 것입니다.

<a name="broadcast-data"></a>
### Broadcast Data
### 브로드캐스트 데이터

When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the event's broadcast payload would be:
이벤트가 브로드캐스트될 때, 이벤트의 모든 `public` 속성들은 자동적으로 시리얼라이즈되어 이벤트의 데이터로 브로드캐스트되고, 자바스크립트 애플리케이션에서 이벤트의 모든 공개적인 데이터에 접근할 수 있도록 해줍니다. 그래서 예를들면, 이벤트가 엘로퀀트 모델을 담고 있는 `$user`라는 퍼블릭 속성 하나를 가지고 있다면, 이벤트의 데이터는 다음과 같이 될 것입니다.

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

However, if you wish to have more fine-grained control over your broadcast payload, you may add a `broadcastWith` method to your event. This method should return the array of data that you wish to broadcast as the event payload:
브로드캐스트 데이터를 더 정교하게 다루고 싶다면 `broadcastWith` 메소드를 사용할 수 있습니다. 이 메소드는 당신이 브로드캐스트하기 원하는 데이터의 배열을 이벤트 데이터로 되돌려 줄 것입니다.

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Broadcast Queue
### 브로드캐스트 대기행렬

By default, each broadcast event is placed on the default queue for the default queue connection specified in your `queue.php` configuration file. You may customize the queue used by the broadcaster by defining a `broadcastQueue` property on your event class. This property should specify the name of the queue you wish to use when broadcasting:
기본적으로, 각 브로드캐스트 이벤트는 `queue.php` 설정 파일에 명시해둔 기본 대기행렬 연결상의 기본 대기행열에 놓이게 될 것입니다. 이벤트 클래스의 `broadcastQueue` 속성을 정의함으로써 대기행렬을 원하는대로 설정할 수 있습니다. 이 속성은 당신이 브로드캐스트할 때 사용하길 원하는 큐의 이름을 지정할 것입니다.

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

<a name="authorizing-channels"></a>
## Authorizing Channels
## 채널 승인하기

Private channels require you to authorize that the currently authenticated user can actually listen on the channel. This is accomplished by making an HTTP request to your Laravel application with the channel name and allowing your application to determine if the user can listen on that channel. When using [Laravel Echo](#installing-laravel-echo), the HTTP request to authorize subscriptions to private channels will be made automatically; however, you do need to define the proper routes to respond to these requests.
비공개 채널은 현재 인증된 사용자가 실제로 채널을 구독할 수 있는지 검사할 것을 요구합니다. 이 검사는 라라벨 애플리케이션에 채널명을 포함한 HTTP 요청을 생성하고 애플리케이션으로 하여금 사용자가 채널을 수신할 수 있는지 판단하게 함으로써 이뤄질 수 있습니다. [Laravel Echo](#installing-laravel-echo)를 쓰면 비공개 채널 구독 승인을 위한 HTTP 요청이 자동으로 생성될 것입니다. 하지만 이러한 요청들에 대한 적절한 응답 라우트는 당신이 정의해줄 필요가 있습니다.

<a name="defining-authorization-routes"></a>
### Defining Authorization Routes
### 승인 라우트 정의하기

Thankfully, Laravel makes it easy to define the routes to respond to channel authorization requests. In the `BroadcastServiceProvider` included with your Laravel application, you will see a call to the `Broadcast::routes` method. This method will register the `/broadcasting/auth` route to handle authorization requests:
고맙게도, 라라벨은 채널 승인 요청에 대한 응답 라우트를 쉽게 정의할 수 있게 해줍니다. 라라벨 애플리케이션에 포함된 `BroadcastServiceProvider`에서 `Broadcast::routes`라는 메소드를 볼 수 있을 것입니다. 이 메소드가 승인 요청을 처리하는 라우트인 `/broadcasting/auth`를 등록해 줄 것입니다.

    Broadcast::routes();

The `Broadcast::routes` method will automatically place its routes within the `web` middleware group; however, you may pass an array of route attributes to the method if you would like to customize the assigned attributes:
`Broadcast::routes` 메소드는 자동적으로 라우트를 `web` 미들웨어 그룹에 위치시킬 것입니다. 그렇지만 할당된 속성들을 커스터마이즈하기 위해서는 메소드에 라우트 속성 배열을 전달해줘야 합니다.

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### Defining Authorization Callbacks
### 승인 콜백 정의하기

Next, we need to define the logic that will actually perform the channel authorization. Like defining the authorization routes, this is also done in the `boot` method of the `BroadcastServiceProvider`. In this method, you may use the `Broadcast::channel` method to register channel authorization callbacks:
이제 실제로 채널 승인을 수행하는 로직을 정의하는 일이 남았습니다. 승인 라우트를 정의하는 것과 같이, 이것도 `BroadcastServiceProvider`의 `boot`메소드를 이용해서 할 수 있습니다. 이 메소드에서 채널 승인 콜백을 등록하기 위해 `Broadcast::channel` 을 사용할 수 있습니다.

    Broadcast::channel('order.*', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.
`channel` 메소드는 두 개의 인수(채널명과 사용자가 채널을 수신하도록 승인되었는지 여부를 나타내는 `true` 혹은 `false` 값을 리턴하는 콜백)를 받습니다.

All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `*` character to indicate that the "ID" portion of the channel name is a wildcard.
모든 승인 콜백은 현재 인증된 사용자를 첫번째 인수로 받고, 다음 인수로 추가적인 와일드카드 파라미터들을 받습니다. 이 예제에서, 채널명의 "ID" 부분을 가리키기 위해 `*`를 사용하는 것이 와일드 카드입니다.


<a name="broadcasting-events"></a>
## Broadcasting Events
## 이벤트 브로드캐스트하기

Once you have defined an event and marked it with the `ShouldBroadcast` interface, you only need to fire the event using the `event` function. The event dispatcher will notice that the event is marked with the `ShouldBroadcast` interface and will queue the event for broadcasting:
이벤트를 정의하고 `ShouldBroadcast` 인터페이스를 이용해 표시했다면, `event` 기능을 이용해서 이벤트를 발생시키기만 하면 됩니다. 이벤트 디스패쳐가 이벤트가 `ShouldBroadcast` 인터페이스를 이용해 표시되었음을 알리고 브로드캐스트하기 위해 이벤트를 대기행렬에 넣을 것입니다.

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Only To Others
### 오직 다른이에게만

When building an application that utilizes event broadcasting, you may substitute the `event` function with the `broadcast` function. Like the `event` function, the `broadcast` function dispatches the event to your server-side listeners:
이벤트 브로드캐스팅을 활용하는 애블리케이션을 만들 때, `event` 기능을 `broadcast` 기능으로 대체할 수 있습니다. `event` 기능과 같이 `broadcast` 기능도 서버단의 리스너들에게 이벤트를 보냅니다.

    broadcast(new ShippingStatusUpdated($update));

However, the `broadcast` function also exposes the `toOthers` method which allows you to exclude the current user from the broadcast's recipients:
그렇지만, `broadcast` 기능은 브로드캐스트 수신자에서 현재 사용자를 제외시켜주는 `toOthers` 메소드를 노출합니다.

    broadcast(new ShippingStatusUpdated($update))->toOthers();

To better understand when you may want to use the `toOthers` method, let's imagine a task list application where a user may create a new task by entering a task name. To create a task, your application might make a request to a `/task` end-point which broadcasts the task's creation and returns a JSON representation of the new task. When your JavaScript application receives the response from the end-point, it might directly insert the new task into its task list like so:
언제 `toOthers` 메소드를 쓰면 좋은지 더 잘 이해하기 위해 할일 이름을 입력함으로써 새로운 할일 을 생성하는 할일 목록 애플리케이션을 상상해봅시다.할일을 생성하기 위해 애플리케이션은 아마도 할일이 생성되었음을 브로드캐스트하고 새 할일의 JSON 표현을 되돌려주는 `/task` 라는 엔드포인트에 요청을 보낼 것입니다. 엔드포인트로부터 자바스크립트 애플리케이션이 응답을 받으면, 다음과 같이 즉각적으로 할일 목록에 새 할일을 추가할 것입니다.

    this.$http.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

However, remember that we also broadcast the task's creation. If your JavaScript application is listening for this event in order to add tasks to the task list, you will have duplicate tasks in your list: one from the end-point and one from the broadcast.
그러나, 할일이 생성되었음을 브로드캐스트도 한다는 것을 기억하세요. 만약 자바스크립트 애플리케이션이 할일 목록에 새 할일을 추가하기 위해 이 이벤트를 수신하고 있다면 하나는 엔드 포인트를 통해, 다른 하나는 브로드캐스트를 통해 할일 목록에 할일이 중복으로 생성될 것입니다.

You may solve this by using the `toOthers` method to instruct the broadcaster to not broadcast the event to the current user.
현재 사용자에게는 이벤트를 브로드캐스트 하지 않도록 브로드캐스터에 지시하기 위해 `toOthers` 메소드를 사용함으로써 이러한 문제를 해결할 수 있습니다.

#### Configuration
#### 설정

When you initialize a Laravel Echo instance, a socket ID is assigned to the connection. If you are using [Vue](https://vuejs.org) and Vue Resource, the socket ID will automatically be attached to every outgoing request as a `X-Socket-ID` header. Then, when you call the `toOthers` method, Laravel will extract the socket ID from the header and instruct the broadcaster to not broadcast to any connections with that socket ID.
라라벨 에코 인스턴스를 초기화 할 때, 커넥션에 소켓 ID 가 할당 됩니다. 만약 당신이 [Vue](https://vuejs.org)와 Vue 리소스를 사용하고 있다면, 모든 외부로 나가는 리퀘스트에 소켓 ID가 `X-Socket-ID` 헤더로 자동으로 첨부될 것입니다. 이후, `toOthers` 메소드를 호출하면, 라라벨은 헤더에서 소켓 ID를 추출해서 브로드캐스터로 하여금 해당 소켓 ID를 가지고 있는 모든 커넥션에 브로드캐스트 하지 않도록 지시할 것입니다.

If you are not using Vue and Vue Resource, you will need to manually configure your JavaScript application to send the `X-Socket-ID` header. You may retrieve the socket ID using the `Echo.socketId` method:
만일 Vue와 Vue 리소스를 사용하지 않는다면, `X-Socket-ID` 헤더를 전송하기 위해 자바스크립트 애플리케이션을 수동으로 설정해주어야 합니다. `Echo.socketId` 메소드를 이용하여 소켓 ID를 받을 수 있습니다.

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Receiving Broadcasts
## 브로드캐스트 받기

<a name="installing-laravel-echo"></a>
### Installing Laravel Echo
### 라라벨 에코 설치하기

Laravel Echo is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by Laravel. You may install Echo via the NPM package manager. In this example, we will also install the `pusher-js` package since we will be using the Pusher broadcaster:
라라벨 에코는 채널을 구독하고 라라벨에 의해 브로드캐스트되는 이벤트를 수신하기 쉽게 해주는 자바스크립트 라이브러리입니다. NPM 패키지 매니저로 에코를 설치할 수 있습니다. 이 예제에서는 Pusher 브로드캐스터를 사용할 것이기 때문에 `pusher-js`도 설치할 것입니다.

    npm install --save laravel-echo pusher-js

Once Echo is installed, you are ready to create a fresh Echo instance in your application's JavaScript. A great place to do this is at the bottom of the `resources/assets/js/bootstrap.js` file that is included with the Laravel framework:
에코가 인스톨되고나면, 애플리케이션의 자바스크립트에서 새로운 에코 인스턴스를 생성할 준비가 완료됩니다. 새로운 에코 인스턴스를 생성하기 좋은 위치는 라라벨 프레임워크와 함께 인클루드되는 `resources/assets/js/bootstrap.js` 파일의 하단입니다.

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

When creating an Echo instance that uses the `pusher` connector, you may also specify a `cluster` as well as whether the connection should be encrypted:
`pusher` 커넥터를 사용하는 에코 인스턴스를 만들 때에는, `cluster`와 커넥션이 암호화 되어야 하는지 여부도 명시할 수 있습니다.

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### Listening For Events
### 이벤트 수신하기

Once you have installed and instantiated Echo, you are ready to start listening for event broadcasts. First, use the `channel` method to retrieve an instance of a channel, then call the `listen` method to listen for a specified event:
에코를 인스톨했다면, 이벤트 브로드캐스트를 수신하기 시작할 준비가 된 것입니다. 첫째로, 채널 인스턴스를 받기 위해 `channel` 메소드를 사용한 후, 특정 이벤트를 수신하기 위해 `listen` 메소드를 호출합니다.

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

If you would like to listen for events on a private channel, use the `private` method instead. You may continue to chain calls to the `listen` method to listen for multiple events on a single channel:
비공개 채널에서 이벤트를 수신하고 싶으면, `private` 메소드를 사용하세요. 하나의 채널에있는 여러 이벤트를 수신하고 싶으면 `listen` 메소드를 연결해서 사용할 수 있습니다.

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="namespaces"></a>
### Namespaces
### 네임스페이스

You may have noticed in the examples above that we did not specify the full namespace for the event classes. This is because Echo will automatically assume the events are located in the `App\Events` namespace. However, you may configure the root namespace when you instantiate Echo by passing a `namespace` configuration option:
위 예제에서 이벤트 클래스의 풀 네임스페이스를 명시하지 않았다는 것을 알아차리셨을지 모르겠습니다. 이는 에코가 자동적으로 이벤트들이 `App\Events` 네임스페이스에 위치하고 있다고 가정하기 때문입니다. 에코를 초기화 할 때 `namespace` 설정 옵션을 넘겨주면 루트 네임스페이스를 변경할 수 있습니다.

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

Alternatively, you may prefix event classes with a `.` when subscribing to them using Echo. This will allow you to always specify the fully-qualified class name:
대안적으로, 에코를 이용하여 이벤트를 구독할 때 이벤트 클래스 앞에 `.`를 붙일 수 있습니다. 이렇게 하면 항상 정규화된 클래스명을 명시할 수 있습니다.

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence Channels
## 프레젠스 채널

Presence channels build on the security of private channels while exposing the additional feature of awareness of who is subscribed to the channel. This makes it easy to build powerful, collaborative application features such as notifying users when another user is viewing the same page.
프레젠스 채널은 비공개 채널에 누가 채널을 구독하고 있는지 알려주는 기능을 추가해줍니다. 프레젠스 채널은 다른 사용자가 같은 페이지를 보고 있을 때 알림을 보내는 것과 같은 강력하고, 협동적인 애플리케이션을 만들기 쉽게 해줍니다.

<a name="joining-a-presence-channel"></a>
### Authorizing Presence Channels
### 프레젠스 채널 승인하기

All presence channels are also private channels; therefore, users must be [authorized to access them](#authorizing-channels). However, when defining authorization callbacks for presence channels, you will not return `true` if the user is authorized to join the channel. Instead, you should return an array of data about the user.
모든 프레젠스 채널은 비공개 채널이기도 합니다. 그러므로 사용자는 당연히 프레젠스에 접근하기 위해 승인을 받아야 합니다[authorized to access them](#authorizing-channels). 그렇지만, 프레젠스 채널을 위한 승인 콜백을 정의할 때, 채널에 들어오는 것을 승인하기 위해 `true`를 되돌려주도록 하는 대신 사용자의 데이터 배열을 되돌려 주어야 합니다.

The data returned by the authorization callback will be made available to the presence channel event listeners in your JavaScript application. If the user is not authorized to join the presence channel, you should return `false` or `null`:
승인 콜백에 의해 되돌려받는 데이터는 자바스크립트 애플리케이션의 프레젠스 채널 이벤트 수신자들이 사용할 수 있게 될 것입니다. 만약 사용자가 프레젠스 채널에 들어가는 것이 승인되지 않았으면, `false`나 `null`을 되돌려 주어야 합니다.

    Broadcast::channel('chat.*', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Joining Presence Channels
### 프레젠스 채널에 들어가기

To join a presence channel, you may use Echo's `join` method. The `join` method will return a `PresenceChannel` implementation which, along with exposing the `listen` method, allows you to subscribe to the `here`, `joining`, and `leaving` events.
프레젠스 채널에 들어가기 위해 에코의 `join` 메소드를 사용할 수 있습니다. `join` 메소드는 `listen` 메소드를 노출하고 `here`, `joining` 그리고 `leaving` 이벤트를 구독할 수 있게 해주는 `PresenceChannel`을 되돌려줄 것입니다.

    Echo.join('chat.' + roomId)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

The `here` callback will be executed immediately once the channel is joined successfully, and will receive an array containing the user information for all of the other users currently subscribed to the channel. The `joining` method will be executed when a new user joins a channel, while the `leaving` method will be executed when a user leaves the channel.
`here` 콜백은 채널이 성공적으로 들어가지면 즉각적으로 실행될 것이고, 현재 채널을 구독하고 있는 다른 모든 사용자들의 정보를 담은 배열을 수신하게 됩니다. `joining` 메소드는 새로운 사용자가 채널에 들어오면 실행될 것이고, `leaving` 메소드는 사용자가 떠나면 실행될 것입니다.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting To Presence Channels
### 프레젠스 채널에 브로드캐스트하기

Presence channels may receive events just like public or private channels. Using the example of a chatroom, we may want to broadcast `NewMessage` events to the room's presence channel. To do so, we'll return an instance of `PresenceChannel` from the event's `broadcastOn` method:
프레젠스 채널 역시 공개 채널이나 비공개 처널과 마찬가지로 이벤트를 받을 수 있습니다. 채팅방의 예를 사용해보면, 방의 프레젠스 채널에 `NewMessage` 이벤트를 브로드캐스트 하길 원할 수 있습니다. 그러기 위해서는 이벤트의 `broadcastOn` 메소드에서 `PresenceChannel` 인스턴스를 되돌려 줄 것입니다.

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Like public or private events, presence channel events may be broadcast using the `broadcast` function. As with other events, you may use the `toOthers` method to exclude the current user from receiving the broadcast:
공개 혹은 비공개 이벤트처럼, 프레젠스 채널 이벤트도 `broadcast` 기능을 이용해 브로드캐스트됩니다. 다른 이벤트와 마찬가지로 현재 사용자를 브로드캐스트 받지 않도록 제외하기 위해서 `toOthers` 메소드를 사용할 수 있습니다.

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

You may listen for the join event via Echo's `listen` method:
에코의 `listen` 메소드를 통해 조인 이벤트를 수신할 수 있습니다.

    Echo.join('chat.' + roomId)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="notifications"></a>
## Notifications
## 알림

By pairing event broadcasting with [notifications](/docs/{{version}}/notifications), your JavaScript application may receive new notifications as they occur without needing to refresh the page. First, be sure to read over the documentation on using [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).
이벤트 브로드캐스팅을 [notifications](/docs/{{version}}/notifications)과 엮음으로써, 당신의 자바스크립트 애플리케이션은 페이지를 새로고침 할 필요없이 새로운 알림을 받을 수 있습니다. 우선 [브로드캐스트 알림 채널](/docs/{{version}}/notifications#broadcast-notifications)을 읽으세요.

Once you have configured a notification to use the broadcast channel, you may listen for the broadcast events using Echo's `notification` method. Remember, the channel name should match the class name of the entity receiving the notifications:
브로드캐스트 채널을 사용하는 것으로 알림을 설정하고나면, 에코의 `notification`메소드를 이용해 브로드캐스트 이벤트를 수신할 수 있습니다. 채널명은 알림을 받는 엔티티의 클래스명과 동일해야 합니다.

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

In this example, all notifications sent to `App\User` instances via the `broadcast` channel would be received by the callback. A channel authorization callback for the `App.User.*` channel is included in the default `BroadcastServiceProvider` that ships with the Laravel framework.
이 예제에서 `broadcast` 채널을 통해 `App\User` 인스턴스에 보내지는 모든 알림들은 콜백에 의해 받아집니다. `App.User.*` 채널에 대한 채널 승인 콜백이 라라벨 프레임워크와 함께 제공되는 기본 `BroadcastServiceProvider` 에 포함됩니다.