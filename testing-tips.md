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

![left](https://upload.wikimedia.org/wikipedia/commons/a/a1/Experts_Expect_the_Unexpected._Nubra.jpg)

## 2

### Know whether you expect pass/fail

---

![left](https://s-media-cache-ak0.pinimg.com/736x/fe/cb/60/fecb607611f43137318d2dd18231b5e7.jpg)

## 3

### ~~Trust, but~~ verify

---

> Never trust a test that you haven't seen fail.
-- Craig Demyanovich (maybe...probably not)

---

> Never say never.
-- Someone more famous than me

^ But why couldn't I find the source?

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

![left](http://animatedheaven.weebly.com/uploads/5/0/1/3/50138243/checklist.png)

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

![left](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Russian-Matroshka_no_bg.jpg/600px-Russian-Matroshka_no_bg.jpg)

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

```ruby
require 'test_helper'

class Api::V1::DealersControllerTest < ActionController::TestCase
  fixtures :organizations, :account_statuses, :dealer_statuses

  context 'POST /authenticate' do
    setup do
      @controller.stubs(:authenticate_request => true)
    end

    context "for active dealer" do
      setup do
        @org = organizations(:comcast)
        @dealer = FactoryGirl.create(:active_ready_dealer, :comcast_affiliate_id => '121500')
        @account = FactoryGirl.create(:dealer_account, :dealer => @dealer, :organization => @org)
        @dealer.locations << FactoryGirl.build(:location, :account => @account, :dealer => @dealer, :photos => [FactoryGirl.build(:location_photo)])
        @dealer.save!
      end

      should "authenticate if password is correct" do
        post_authenticate @account.user_name, @account.password
        assert_response :success
      end

      should "return correct json" do
        post_authenticate @account.user_name, @account.password
        loc_data = []
        @dealer.locations.each {|l| loc_data += [{:address => l.contact.address.address_line_1, :affid => l.affid }]}
        response = JSON.parse(@response.body)

        assert_equal 'correct', response['status']
        assert_equal @dealer.alias_name, response['name']
        assert_equal @dealer.cid, response['cid']
        assert_equal @dealer.oid, response['oid']
        assert_equal @dealer.comcast_affiliate_id, response['comcast_affiliate_id']
        assert_equal @dealer.twc_affiliate_id, response['twc_affiliate_id']
        assert_equal @org.name, response['organization']
        assert_equal @dealer.id, response['id']
        assert_equal @dealer.national, response['national']
        assert_equal @dealer.pci_compliant, response['pci_compliant']
        assert_equal @dealer.can_check_positive_id, response['can_check_positive_id']
        assert_equal @dealer.can_sell_twc_intelligent_home_products, response['can_sell_twc_intelligent_home_products']
        assert_equal @dealer.can_sell_xhs_products, response['can_sell_xhs_products']
        assert_equal @dealer.cpos_trained, response['cpos_trained']
        assert_equal loc_data.first[:address], response['locations'].first['address']
        assert_equal loc_data.first[:affid], response['locations'].first['affid']
      end

      should "return the dealers' oid with trailing whitespace removed" do
        @dealer.update_attribute(:oid, "123456 ")
        post_authenticate @account.user_name, @account.password
        response = JSON.parse(@response.body)
        assert_equal response['oid'], "123456"
      end

      should "not authenticate if password is missing" do
        post_authenticate @account.user_name, nil
        assert_response 403
      end

      should "not authenticate if password is incorrect" do
        post_authenticate @account.user_name, 'not_correct'
        assert_response 403
      end

      should "return response for dealer with location without contact" do
        @dealer.locations.first.contact = nil
        @dealer.save
        post_authenticate @account.user_name, @account.password
        assert_response :success

        loc_data = []
        @dealer.locations.each {|l| loc_data += [{:address => nil, :affid => l.affid }]}
        expected = { :status => :correct,
                     :name => @dealer.alias_name,
                     :id => @dealer.id,
                     :cid => @dealer.cid,
                     :oid => @dealer.oid,
                     :comcast_affiliate_id => @dealer.comcast_affiliate_id,
                     :twc_affiliate_id => @dealer.twc_affiliate_id,
                     :national => @dealer.national,
                     :pci_compliant => @dealer.pci_compliant,
                     :can_check_positive_id => @dealer.can_check_positive_id?,
                     :can_sell_twc_intelligent_home_products => @dealer.can_sell_twc_intelligent_home_products?,
                     :can_sell_xhs_products => @dealer.can_sell_xhs_products?,
                     :cpos_trained => @dealer.cpos_trained?,
                     :organization => @org.name,
                     :locations => loc_data,
        }.to_json
        assert_equal expected, @response.body
      end

      should "return response for dealers with 1 location without contact" do
        account = FactoryGirl.build(:dealer_account, :organization => @org)
        location = FactoryGirl.build(:location, :account => account, :dealer => @dealer, :contact => nil)
        location.save
        post_authenticate @account.user_name, @account.password
        assert_response :success

        loc_data = []
        @dealer.locations.each {|l| loc_data += [{:address => (l.contact.present? ? l.contact.address.address_line_1 : nil), :affid => l.affid }]}
        expected = { :status => :correct,
                     :name => @dealer.alias_name,
                     :id => @dealer.id,
                     :cid => @dealer.cid,
                     :oid => @dealer.oid,
                     :comcast_affiliate_id => @dealer.comcast_affiliate_id,
                     :twc_affiliate_id => @dealer.twc_affiliate_id,
                     :national => @dealer.national,
                     :pci_compliant => @dealer.pci_compliant,
                     :can_check_positive_id => @dealer.can_check_positive_id?,
                     :can_sell_twc_intelligent_home_products => @dealer.can_sell_twc_intelligent_home_products?,
                     :can_sell_xhs_products => @dealer.can_sell_xhs_products?,
                     :cpos_trained => @dealer.cpos_trained?,
                     :organization => @org.name,
                     :locations => loc_data,
        }.to_json
        assert_equal expected, @response.body
      end

    end

    context "when dealer is not found by username" do
      should "not render a 500 and not authenticate" do
        post_authenticate 'notARealName', 'testpass'
        assert_response 403
      end
    end

    context "for inactive dealer" do
      setup do
        org = organizations(:comcast)
        @dealer = FactoryGirl.build(:inactive_dealer)
        @account = FactoryGirl.build(:dealer_account, :dealer => @dealer, :account_status_id => AccountStatus.inactive.id, :organization => org)
        @dealer.locations << FactoryGirl.build(:location, :account => @account, :dealer => @dealer)
        @dealer.save
      end

      should "not authenticate even if password is correct" do
        post_authenticate @account.user_name, @account.password
        assert_response 403
      end
    end

    context "linked accounts" do
      setup do
        @dealer = FactoryGirl.build(:active_dealer)
        @dealer.pci_compliance_declarations.build :transmitted_at => DateTime.current, :document_key => 'abc123', :signed_at => DateTime.current
        account = FactoryGirl.build(:dealer_account, :dealer => @dealer, :organization => organizations(:comcast))
        @dealer.locations << FactoryGirl.build(:location, :account => account, :dealer => @dealer, :photos => [FactoryGirl.build(:location_photo)])
        @dealer.save!
        # Ensure role_type_id is set first, since Account depends on it to
        # determine what to validate
        linked_account = @dealer.linked_accounts.build
        linked_account.role_type_id = AccountRole.linked_account.id
        linked_account.first_name = 'Fede'
        linked_account.last_name = 'Zuppa'
        linked_account.user_name = 'fede'
        linked_account.password = '123456'
        linked_account.email = 'fzuppa@novalid.com'
        linked_account.account_status_id = AccountStatus.active.id
        linked_account.save!
      end

      should "find the associated dealer" do
        post_authenticate 'fede', '123456'
        body = JSON.parse(@response.body)
        assert_equal @dealer.alias_name, body['name']
        assert_equal @dealer.oid, body['oid']
      end

      should "not be able to authenticate with not existing user" do
        post_authenticate 'fede132', '123456'
        assert_response 403
      end

      should "not be able to authenticate with wrong password" do
        post_authenticate 'fede', '876543'
        assert_response 403
      end
    end
  end

  context 'GET /show with invalid id' do
    setup do
      @controller.stubs(:authenticate_request => true)
    end

    should 'respond with 404 (not found) for unknown id' do
      get :show, :id => '0', :format => 'json'
      assert_response :not_found
    end
  end

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

    context "JSON request" do
      setup do
        get :show, :id => @dealer.id, :format => 'json'
      end

      should respond_with :success

      should "be json response" do
        assert_equal "application/json", @response.content_type
      end

      should "have location info" do
        assert_match @location.affid, @response.body
      end

      should "not have virtual location info" do
        assert_no_match Regexp.new(@virtual_location.affid), @response.body
      end

      context "hierarchy" do
        setup do
          @decoded_response = JSON::parse(@response.body)
        end

        should "include dealer" do
          assert @decoded_response.keys.include? "dealer"
        end

        should "include locations" do
          assert @decoded_response["dealer"].keys.include? "locations"
        end

        should "have contacts for locations" do
          assert @decoded_response["dealer"]["locations"].first.keys.include? "contact"
        end

        should "have an address for contacts for locations" do
          assert @decoded_response["dealer"]["locations"].first["contact"].keys.include? "address"
        end

        should "include representative" do
          assert @decoded_response["dealer"].keys.include? "representative"
        end

        should "include contacts" do
          assert @decoded_response["dealer"].keys.include? "contacts"
        end

        should "include status" do
          assert @decoded_response["dealer"].keys.include? "status"
        end

        should 'include flag indicating whether dealer is national' do
          assert_equal false, @decoded_response['dealer']['national']
        end

        should 'include flag indicating whether dealer is PCI compliant' do
          assert_equal true, @decoded_response['dealer']['pci_compliant']
        end

        should 'include flag indicating whether dealer can check Positive ID' do
          assert_equal false, @decoded_response['dealer']['can_check_positive_id']
        end

        should 'include flag indicating whether dealer can sell XHS products' do
          assert_equal false, @decoded_response['dealer']['can_sell_xhs_products']
        end

        should 'include flag indicating whether dealer is CPos trained' do
          assert_equal false, @decoded_response['dealer']['cpos_trained']
        end

        should 'include twc_sales_id' do
          assert_equal 'foo', @decoded_response['dealer']['twc_sales_id']
        end

        # NOTE: would be nice to ensure that sensitive attributes are
        # globally excluded.
        ['password_digest'].each do |attr|
          should "exclude #{attr}" do
            assert_false @decoded_response['dealer']['representative'].has_key? attr
          end
        end
      end
    end
  end

  def post_authenticate(user_name, password)
    post :authenticate, :format => 'json', :user_name => user_name, :password => password
  end
end
```

---

#### TODO: Instead, use helper functions...

---

![left](https://upload.wikimedia.org/wikipedia/commons/thumb/0/04/Mud_and_water_suspension_and_clear_water.JPG/1024px-Mud_and_water_suspension_and_clear_water.JPG)

## 8

### Order assertions for helpful error messages

---

#### TODO: show example from PAPS

---

![left](https://c2.staticflickr.com/2/1442/24212100174_455e70ce34_b.jpg)

## 9

### Not just 1:1

---

#### TODO: show example

---

![left](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Carrots_of_many_colors.jpg/635px-Carrots_of_many_colors.jpg)

## 10

### Data-driven

---

#### TODO: show example

---

![](https://youtu.be/XuzpsO4ErOQ?t=48s)

---

![left](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1c/Haskell-Logo.svg/800px-Haskell-Logo.svg.png)

## 11

### Generative testing

---

#### Example libraries

* [QuickCheck](https://hackage.haskell.org/package/QuickCheck)
* [test.check](https://github.com/clojure/test.check)
* [clojure.spec](http://clojure.org/about/spec)

---

#### Example (clojure.spec)

```clojure
```

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
