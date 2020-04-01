import React, { Component } from "react";
import { Card } from "antd";
import "./Help.css";
import lb_diagram from "./lb-diagram-new.png";

const { Meta } = Card;

const help_loadbalance = {
  name: "Virtual IP",
  title: `
    When a client queries for “www.yourcompany.com”, they get an IP address,
    of course. In many cases if the site is served by a load balancer or application
    delivery controller that IP address is a virtual IP address.
    That simply means the IP address is not tied to a specific host.
    It is kind of floating out there, waiting for requests.
    In the case of the above example, the client obtains 192.0.2.1 as the Virtual
    IP. He does not see the hosts' IPs behind.
  `,
  picture: lb_diagram
};
const help_vip = {
  name: "Virtual IP",
  title: `
    When a client queries for “www.yourcompany.com”, they get an IP address,
    of course. In many cases if the site is served by a load balancer or application
    delivery controller that IP address is a virtual IP address.
    That simply means the IP address is not tied to a specific host.
    It is kind of floating out there, waiting for requests.
    In the case of the above example, the client obtains 192.0.2.1 as the Virtual
    IP. He does not see the hosts' IPs behind.
  `,
  picture: lb_diagram
};
const help_snat = {
  name: "SNAT IP",
  title: `
    When a client sends information to the web cluster through a VIP, they
    will reach the loadbalancer. In the example above, the client will be going
    outbound to 192.0.2.1. Once the information is reached the loadbalancer, it will
    translate your source IP from 198.18.0.1 to 172.18.1.1! The web cluster no longer
    see the source IP of the client (198.18.0.1).
  `,
  picture: lb_diagram
};

const help_active = {
  name: "Active-Passive vs Active-Active",
  title: `
    Active-Passive refers to imbalanced loadbalancing. It means that it will always
    choose a certain server. For simplicity in the example above, green refers to
    active and orange refers to passive. When the client connects to the web cluster,
    the loadbalancer will choose to send traffic to 172.16.1.11 (green) as long as
    172.16.1.11 is available. 172.16.1.12 (orange) will not have traffic as long as 172.16.1.11 (green)
    is available. Active-Active will loadbalance to any servers.
  `,
  picture: lb_diagram
};

class HelpPage extends Component {
  state = {
    help_dict: {
      loadbalance: help_loadbalance,
      vip: help_vip,
      snat: help_snat,
      active: help_active
    }
  };
  render() {
    const help = this.state.help_dict[this.props.keyword];
    return (
      <div>
        <Card cover={<img alt="example" src={help.picture} />}>
          <Meta title={help.name} description={help.title} />
        </Card>
      </div>
    );
  }
}

export default HelpPage;
