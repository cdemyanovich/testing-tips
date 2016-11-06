theme: Plain Jane, 2

![](https://c1.staticflickr.com/3/2388/2376598010_9543d6a12b_b.jpg)

# Testing Tips

---

![left](http://theprepperproject.com/wp-content/uploads/2013/12/BrokenHammer.jpg)

## 1

### Ensure your tools work

---

```cs
using System;
using Xunit;

namespace Foo.Tests {
    public class DoSomethingTest {
        [Fact]
        public void It_Works() {
            Assert.Equal(1, 1);
        }
    }
}
```

---

![left](https://upload.wikimedia.org/wikipedia/commons/a/a1/Experts_Expect_the_Unexpected._Nubra.jpg)

## 2

### Predict the future

---

![left fit](https://s-media-cache-ak0.pinimg.com/736x/fe/cb/60/fecb607611f43137318d2dd18231b5e7.jpg)

## 3

### ~~Trust, but~~ verify

---

> Never trust a test that you haven't seen fail.
-- Anonymous

---

```cs
using System;
using Xunit;

namespace Foo.Tests {
    public class DoSomethingTest {
        [Fact]
        public void It_Works() {
            Assert.Equal(1, 2);
        }
    }
}
```

---

![left](https://c2.staticflickr.com/6/5246/5379549514_f64ef63a0e_o.jpg)

## 4

### Delete tests

---

![fit](https://upload.wikimedia.org/wikipedia/commons/thumb/3/31/ProhibitionSign2.svg/600px-ProhibitionSign2.svg.png)

```cs
using System;
using Xunit;

namespace Foo.Tests {
    public class DoSomethingTest {
        [Fact]
        public void It_Works() {
            Assert.Equal(1, 1);
        }
    }
}
```

---

>...perfection is finally attained not when there is no longer anything to add, but when there is no longer anything to take away...
-- Antoine de Saint Exupéry

^ Terres des Hommes (1939), Ch. III: L'Avion, p. 60

---

![left fit](http://animatedheaven.weebly.com/uploads/5/0/1/3/50138243/checklist.png)

## 5

### Create a test list

---

#### Validate a vehicle

* Vehicle is valid with all required attributes
* Vehicle is invalid without VIN
* Vehicle is invalid without year
* Vehicle is invalid without make
* Vehicle is invalid without model

etc.

^ mental/paper/pending tests

---

![left](corinne-ireland-cliff.jpg)

## 6

### Test at the boundaries

---

#### Producing a rate requires vehicles

* Rate with 0 vehicles
* Rate with 1 vehicle
* Rate with n vehicles
* Rate with max - 1 vehicles
* Rate with max vehicles
* Rate with max + 1 vehicles

---

![left fit](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Russian-Matroshka_no_bg.jpg/600px-Russian-Matroshka_no_bg.jpg)

## 7

### Avoid nested/shared setup

^ Matryoshka doll

---

```ruby
        should "include dealer" do
          assert @decoded_response.keys.include? "dealer"
        end
```

^ `dealers_controller_test.rb:237`

---

```ruby
      context "hierarchy" do
        setup do
          @decoded_response = JSON::parse(@response.body)
        end
```

^ `dealers_controller_test.rb:231:234`

---


```ruby
    context "JSON request" do
      setup do
        get :show, :id => @dealer.id, :format => 'json'
      end
```

^ `dealers_controller_test.rb:212:215`

---

```ruby
  context 'GET /show with valid id' do
    fixtures :organizations
    setup do
      organization = organizations(:comcast)
      @controller.stubs(:authenticate_request => true)
      @controller.stubs(:current_organization => organization)

      @dealer = FactoryGirl.create(:active_ready_dealer, :twc_sales_id => 'foo')
      @account = FactoryGirl.create(:dealer_account, :dealer => @dealer, :organization => organization)
      @location = FactoryGirl.build(:location, :dealer => @dealer, :account => @accoun, :photos => [FactoryGirl.build(:location_photo)])
      @dealer.locations << @location
      @virtual_location = FactoryGirl.build(:virtual_location, :dealer => @dealer, :account => @account)
      @dealer.locations << @virtual_location
      @dealer.save!
      Dealer.any_instance.stubs(:division).returns('test')
    end
```

^ `dealers_controller_test.rb:195:210`

---

```ruby
  context 'POST /authenticate' do
    setup do
      @controller.stubs(:authenticate_request => true)
    end
```

^ `dealers_controller_test.rb:6:9`

---

237 - 233 = 4

233 - 214 = 19

214 - 202 = 12

202 - 8 = 196

__4 + 19 + 12 + 196 = 231__

---

![fit](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Adhesive_bandage_drawing_nevit.svg/1024px-Adhesive_bandage_drawing_nevit.svg.png)

* Bookmarks
* Code folding
* Comments
* etc.

---

#### Instead...

Refactor: Extract method

```ruby
def show_dealer(id)
  get :show, :id => id, :format => 'json'
  JSON::parse(@response.body)
end
```

```ruby
response = show_dealer(@dealer.id)
assert response.keys.include? "dealer"
```

^ design opportunity (for tests and production code)
^ maybe compose these small methods, e.g., other response formats
^ extracted methods/classes can move to domain

---

![left fit](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/American-Automobile-Association-Logo.svg/800px-American-Automobile-Association-Logo.svg.png)

## 8

### Obey the "AAA" rule

---

__A__rrange

__A__ct

__A__ssert

---

```ruby
should "include dealer" do
  assert @decoded_response.keys.include? "dealer"
end
```

```ruby
should "include dealer" do
  dealer = create_dealer(...)
  response = show_dealer(dealer.id)
  assert response.keys.include? "dealer"
end
```

---

![left](https://images-na.ssl-images-amazon.com/images/M/MV5BMTYxMzc0NDk1NV5BMl5BanBnXkFtZTgwNTcyMDEyODE@._V1_SY1000_CR0,0,659,1000_AL_.jpg)

## 9

### Not just 1:1

---

>There can be only one!
-- Connor MacLeod, Ramirez, The Kurgan

---

>There can be two or more!
-- Craig Demyanovich

---

#### In practice...

* not that often
* may indicate an opportunity for extracting a class/method

---

![left](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Carrots_of_many_colors.jpg/635px-Carrots_of_many_colors.jpg)

## 10

### Data-driven

---

```cs
using System;
...
using Xunit;

namespace PersonalAutoPolicyService.Tests.V1.Json {
    public class VehicleHistoryJsonTest {
        [Theory]
        [InlineData(AverageAnnualMileageStrategy.OneA, "1A")]
        [InlineData(AverageAnnualMileageStrategy.OneB, "1B")]
        [InlineData(AverageAnnualMileageStrategy.Two, "2")]
        [InlineData(AverageAnnualMileageStrategy.Default, "default")]
        [InlineData(null, null)]
        public void Converts_The_AverageAnnualMileageStrategy(AverageAnnualMileageStrategy? strategy, string jsonValue) {
            Assert.Equal(jsonValue, strategy.ToJson());
        }
    }
}
```

---

![](https://youtu.be/XuzpsO4ErOQ?t=48s)

---

![left fit](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1c/Haskell-Logo.svg/800px-Haskell-Logo.svg.png)

## 11

### Generative testing

---

1. Automates generation of tests
1. Covers large sample of inputs
1. Computes a minimal failing case

---

#### Example libraries

* [QuickCheck](https://hackage.haskell.org/package/QuickCheck)
* [test.check](https://github.com/clojure/test.check)
* [elm-check](https://github.com/TheSeamau5/elm-check)
* [FsCheck](https://fscheck.github.io/FsCheck/)
* [clojure.spec](http://clojure.org/about/spec)

---

#### Example (clojure.spec)[^1]

```clojure
(spec/def ::velocity p/positive-int)
(spec/def ::x p/positive-int)
(spec/def ::y p/positive-int)
(spec/def ::position (spec/keys :req-un [::x ::y]))
(spec/def ::laser (spec/keys :req-un [::position ::velocity]))

(def initial
  {:position {:x 98 :y 216}
   :velocity 0})
```

[^1]: Eric Smith (@paytonrules) in #support-hotline: https://8thlight.slack.com/archives/support-hotline/p1478008717002944

---

# Fin

---

# Image Credits

* ["Experts Expect the Unexpected"](https://commons.wikimedia.org/wiki/File%3AExperts_Expect_the_Unexpected._Nubra.jpg) By John Hill (Own work) [CC BY-SA 4.0 (http://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons
* "Delete buttons on Apple keyboard" by Matt McGee, http://www.carimcgee.com/
* ["Matroshka"](https://commons.wikimedia.org/wiki/File%3ARussian-Matroshka_no_bg.jpg) By Original photo: User:Fanghong Derivative work: User:Gnomz007 [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC-BY-SA-3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons

---

# Image Credits (continued)

* ["Mud and water suspension and clear water"](https://commons.wikimedia.org/wiki/File%3AMud_and_water_suspension_and_clear_water.JPG) By Meganbeckett27 (Own work) [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0)], via Wikimedia Commons
* ["Carrots of many colors"](https://commons.wikimedia.org/wiki/File%3ACarrots_of_many_colors.jpg) By Stephen Ausmus [Public domain], via Wikimedia Commons
* ["Golden Section"](https://www.flickr.com/photos/themarmot/24212100174) By The Marmot

---

# Image Credits (continued)

* ["Haskell Logo"](https://commons.wikimedia.org/wiki/File%3AHaskell-Logo.svg) By Thought up by Darrin Thompson and produced by Jeff Wheeler (Thompson-Wheeler logo on the haskell wiki) [Public domain or Public domain], via Wikimedia Commons
* ["American Automobile Association Logo"](https://commons.wikimedia.org/wiki/File%3AAmerican-Automobile-Association-Logo.svg) See page for author [Public domain], via Wikimedia Commons
