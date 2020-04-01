import React, { Component } from "react";
import { Button, Icon } from "antd";

class ButtonRow extends Component {
  serverRender = () => {
    const { data } = this.props;
    const disabled = this.serverDisabled(data);
    return (
      <div
        style={{
          display: "flex",
          flexWrap: "wrap",
          justifyContent: "space-between"
        }}
      >
        <Button type="secondary" id="1" disabled={true}>
          Back
        </Button>
        <Button type="primary" onClick={this.props.onPreview}>
          Request Preview
        </Button>
        <Button
          type="primary"
          id="0"
          style={{ float: "right" }}
          onClick={this.props.onNext}
          disabled={disabled}
        >
          {disabled ? (
            <Icon type="close-circle" style={{ color: "#f5222d" }} />
          ) : (
            <Icon type="check-circle" />
          )}
          Next
        </Button>
      </div>
    );
  };
  advanceRender = () => {
    const disabled = false;
    return (
      <div
        style={{
          display: "flex",
          flexWrap: "wrap",
          justifyContent: "space-between"
        }}
      >
        <Button type="secondary" id="2" onClick={this.props.onBack}>
          Back
        </Button>
        <Button type="primary" onClick={this.props.onPreview}>
          Request Preview
        </Button>
        <Button
          type="primary"
          id="2"
          style={{ float: "right" }}
          onClick={this.props.onSubmit}
          disabled={disabled}
        >
          {disabled ? (
            <Icon type="close-circle" style={{ color: "#f5222d" }} />
          ) : (
            <Icon type="check-circle" />
          )}
          Submit
        </Button>
      </div>
    );
  };
  serverDisabled = data => {
    if (data.gtm) {
      if (
        Object.values(data.gtm).filter(item => item.wideip.length === 0)
          .length > 0
      ) {
        return true;
      }
    }
    if (data.ltm) {
      if (
        Object.values(data.ltm).filter(item => {
          return (
            item.vs_name.length === 0 ||
            item.vip.length === 0 ||
            item.vipport.length === 0 ||
            item.snatip.length === 0 ||
            Object.values(item.servers).filter(svr => {
              return svr.ipaddress.length === 0 || svr.port.length === 0;
            }).length > 0
          );
        }).length > 0
      ) {
        return true;
      }
    }
    return false;
  };
  buttonRender = () => {
    if (this.props.current === 0) {
      return this.serverRender();
    } else if (this.props.current === 1) {
      return this.advanceRender();
    }
  };
  render() {
    return <div>{this.buttonRender()}</div>;
  }
}

export default ButtonRow;
