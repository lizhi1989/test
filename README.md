import React, { Component } from "react";
import { Switch, Route, Redirect } from "react-router-dom";
import PropTypes from "prop-types";
import { Layout, Icon, message, Row, Col } from "antd";
import { environment, basename, appname } from "../../local_settings";

import { notification } from "antd";

import { histAppPush } from "../../history";

import coreApi from "../../api/coreApi";
import NoDecoButton from "../Common/NoDecoButton";

import Sidebar from "../Wireframe/Sidebar/Sidebar";
import Header from "../Wireframe/Header/Header";
// import Breadcrumb from "../Wireframe/Breadcrumb/Breadcrumb";
// import Footer from '../Wireframe/Footer/Footer'

import LTMCreate from "../LTM/Create/Create";
import LTMUpdate from "../LTM/Update/Update";
import LTMDelete from "../LTM/Delete/Delete";
import LTMSearch from "../LTM/Search/Search";
// import LTMUpload from '../LTM/Upload/Upload'
// import LTMRecord from '../LTM/Record/Record'
import LTMRequest from "../LTM/Request/Request";

import GTMCreate from "../GTM/Create/Create";
import GTMSearch from "../GTM/Search/Search";
import GTMUpdate from "../GTM/Update/Update";
import GTMDelete from "../GTM/Delete/Delete";
// import GTMUpload from '../GTM/Upload/Upload'
// import GTMRecord from '../GTM/Record/Record'
import GTMRequest from "../GTM/Request/Request";
// import Start from '../Start/Start'
import LandingPage from "../LandingPage/LandingPage";
import { AppcodeContext } from "../context";
import styled from "styled-components";

const { Content } = Layout;
const PaddedContent = styled(Content)`
  width: 80%;
  margin: 32px auto;
  @media (max-width: 1600px) and (min-width: 769px) {
    width: 95%;
  }
  @media (max-width: 768px) {
    width: 100%;
  }
`;

const PaddedLayout = styled(Layout)`
  margin-left: 200px;
  @media (max-width: 1600px) and (min-width: 769px) {
    margin-left: 80px;
  }
  @media (max-width: 768px) {
    margin-left: 0px;
  }
`;

const approve_style = {
  color: "#fa8c16"
};

function BlockRoute({ component: Component, ...rest }) {
  return (
    <Route
      {...rest}
      render={props =>
        sessionStorage.block === "true" ? (
          <Redirect
            to={{
              pathname: `${basename}${appname}/block`,
              state: { from: props.location }
            }}
          />
        ) : (
          <Component {...props} />
        )
      }
    />
  );
}

class Main extends Component {
  state = {
    width: 1920,
    height: 1080,
    collapsed: false,
    loading: true,
    error: false,
    app_list: []
  };
  onCollapse = collapsed => {
    this.setState({ collapsed });
  };
  goGTMApprove = () => {
    notification.destroy();
    histAppPush("/gtm/request/?complete=pending");
  };
  goLTMApprove = () => {
    notification.destroy();
    histAppPush("/ltm/request/?complete=pending");
  };
  // componentWillMount = () => {
  //   sessionStorage.setItem('appr_num', 0);
  // }
  updateDimensions = () => {
    this.setState({ width: window.innerWidth, height: window.innerHeight });
  };
  componentWillMount = () => {
    this.updateDimensions();
  };
  componentWillUnmount = () => {
    window.removeEventListener("resize", this.updateDimensions);
  };
  componentDidMount = () => {
    window.addEventListener("resize", this.updateDimensions);
    coreApi.getApprovalNum().then(async response => {
      if (response.status > 300) {
        this.setState({ loading: false, error: true });
      } else {
        const rjson = await response.json();
        sessionStorage.setItem("ltmrequest_num", rjson.ltmrequest);
        sessionStorage.setItem("gtmrequest_num", rjson.gtmrequest);
        coreApi.getBlock().then(async response => {
          if (response.status < 300) {
            const rjson = await response.json();
            sessionStorage.setItem("block", rjson.block);
          } else {
            sessionStorage.setItem("block", false);
          }
        });
        // sessionStorage.setItem("block", rjson.block); // going to do this after that
        if (rjson.ltmrequest > 0 || rjson.gtmrequest > 0) {
          notification.info({
            icon: <Icon type="switcher" />,
            message: "Pending Approvals",
            description: (
              <div>
                {rjson.gtmrequest > 0 && (
                  <div>
                    You have{" "}
                    <NoDecoButton
                      onClick={this.goGTMApprove}
                      style={approve_style}
                    >
                      {rjson.gtmrequest} GTM requests
                    </NoDecoButton>{" "}
                    pending.
                  </div>
                )}
                {rjson.ltmrequest > 0 && (
                  <div>
                    You have{" "}
                    <NoDecoButton
                      onClick={this.goLTMApprove}
                      style={approve_style}
                    >
                      {rjson.ltmrequest} LTM requests
                    </NoDecoButton>{" "}
                    pending.
                  </div>
                )}
              </div>
            )
          });
        }
        this.setState({ loading: false });
        coreApi.getAppcodes().then(async response => {
          if (response.status > 300) {
            message.error("Not able to get appcodes!");
          } else {
            const app_list = await response.json();
            this.setState({ app_list });
          }
        });
      }
    });
  };
  render() {
    const { width } = this.state;
    if (this.state.loading) {
      return (
        <div
          style={{
            position: "relative",
            height: "100vh",
            width: "100vw",
            background: "#91d5ff"
          }}
        >
          <Row
            type="flex"
            justify="center"
            style={{ position: "absolute", top: "30%", width: "100%" }}
          >
            <Col span={24}>
              <div
                style={{
                  display: "flex",
                  justifyContent: "center",
                  justifyItems: "center"
                }}
              >
                <Icon type="loading" style={{ fontSize: 120 }} />
              </div>
            </Col>
            <Col span={24}>
              <div
                style={{
                  display: "flex",
                  justifyContent: "center",
                  justifyItems: "center"
                }}
              >
                <h2
                  style={{
                    fontSize: width > 800 ? 40 : (40 * width) / 800,
                    fontWeight: 600
                  }}
                >
                  Checking Authorization...
                </h2>
              </div>
            </Col>
          </Row>
        </div>
      );
    } else if (this.state.error) {
      return (
        <div
          style={{
            position: "relative",
            height: "100vh",
            width: "100vw",
            background: "#ffa39e"
          }}
        >
          <Row
            type="flex"
            justify="center"
            style={{ position: "absolute", top: "30%", width: "100%" }}
          >
            <Col span={24}>
              <div
                style={{
                  display: "flex",
                  justifyContent: "center",
                  justifyItems: "center"
                }}
              >
                <Icon type="warning" style={{ fontSize: 120 }} />
              </div>
            </Col>
            <Col span={24}>
              <div
                style={{
                  display: "flex",
                  justifyContent: "center",
                  justifyItems: "center"
                }}
              >
                <div style={{ textAlign: "center" }}>
                  <h2
                    style={{
                      fontSize: width > 800 ? 40 : (40 * width) / 800,
                      fontWeight: 600,
                      marginBottom: 0
                    }}
                  >
                    Not Authorized
                  </h2>
                  <div>
                    Please check with AIMS for access via Network Automation
                  </div>
                </div>
              </div>
            </Col>
          </Row>
        </div>
      );
    } else {
      return (
        <Layout style={{ minHeight: "100vh" }}>
          <Sidebar width={width} />
          <PaddedLayout>
            {environment === "development" && <Header />}
            <PaddedContent>
              {/* <Breadcrumb /> */}
              <AppcodeContext.Provider value={this.state.app_list}>
                <Switch>
                  {environment !== "development" && (
                    <Route
                      exact
                      path={`${this.props.match.url}/`}
                      component={LandingPage}
                    />
                  )}
                  <Route
                    path={`${this.props.match.url}/home`}
                    component={LandingPage}
                  />

                  <BlockRoute
                    path={`${this.props.match.url}/ltm/create/:index`}
                    component={LTMCreate}
                  />
                  <BlockRoute
                    path={`${this.props.match.url}/ltm/create/`}
                    component={LTMCreate}
                  />
                  {/* <Route path={`${this.props.match.url}/ltm/upload/`} component={LTMUpload}/> */}

                  <Route
                    path={`${this.props.match.url}/ltm/search/`}
                    component={LTMSearch}
                  />
                  <BlockRoute
                    path={`${this.props.match.url}/ltm/update/:index`}
                    component={LTMUpdate}
                  />
                  <BlockRoute
                    path={`${this.props.match.url}/ltm/delete/:index`}
                    component={LTMDelete}
                  />

                  <Route
                    path={`${this.props.match.url}/ltm/request/`}
                    component={LTMRequest}
                  />

                  <BlockRoute
                    path={`${this.props.match.url}/gtm/create/`}
                    component={GTMCreate}
                  />
                  {/* <Route path={`${this.props.match.url}/gtm/upload/`} component={GTMUpload}/> */}

                  <Route
                    path={`${this.props.match.url}/gtm/search/`}
                    component={GTMSearch}
                  />
                  <BlockRoute
                    path={`${this.props.match.url}/gtm/update/:index`}
                    component={GTMUpdate}
                  />
                  <BlockRoute
                    path={`${this.props.match.url}/gtm/delete/:index`}
                    component={GTMDelete}
                  />

                  <Route
                    path={`${this.props.match.url}/gtm/request/`}
                    component={GTMRequest}
                  />
                  <Route
                    path={`${this.props.match.url}/block`}
                    component={() => (
                      <div>
                        Please ensure that you have no more than 10{" "}
                        <NoDecoButton
                          onClick={() =>
                            histAppPush("/ltm/request/?complete=pending")
                          }
                        >
                          pending/approved requests
                        </NoDecoButton>{" "}
                        in this portal.
                      </div>
                    )}
                  />
                  <Route
                    component={() => (
                      <h1 style={{ fontWeight: "700", marginBottom: 5 }}>
                        <Icon
                          type="close-circle-o"
                          style={{ marginRight: 10 }}
                        />
                        Page Not Found
                      </h1>
                    )}
                  />
                </Switch>
              </AppcodeContext.Provider>
            </PaddedContent>
          </PaddedLayout>
        </Layout>
      );
    }
  }
}

Main.propTypes = {
  match: PropTypes.object.isRequired
};

export default Main;
