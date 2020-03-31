import React from "react";
import { Tag, Tree, Icon } from "antd";

const TreeNode = Tree.TreeNode;

const colorTag = {
  CREATE: "blue",
  DELETE: "red",
  UPDATE: "red",
  OWNERSHIP: "blue"
};

// const RequestShortDesc = ({ record }) => (
//   <div>
//     {record.items.map(item => (
//       <div key={item.id}>
//         {Object.values(item.wideip).map((wip, wip_num) => (
//           <div key={wip.name}>
//             <Tree showLine defaultExpandAll={true}>
//               <TreeNode
//                 title={
//                   <div>
//                     <Tag color={colorTag[item.CRUD]} style={{ cursor: "text" }}>
//                       {item.CRUD}
//                     </Tag>
//                     {wip.name}
//                   </div>
//                 }
//                 key={wip.num}
//               >
//                 {Object.values(wip.pools).map(pool => (
//                   <TreeNode title={pool.name} key={`${wip.num}-${pool.id}`}>
//                     {pool.servers.map(svr => (
//                       <TreeNode
//                         title={
//                           svr.pub_destination ? (
//                             <span>
//                               {svr.destination} (
//                               <span style={{ color: "red" }}>
//                                 {svr.pub_destination}
//                               </span>
//                               )
//                             </span>
//                           ) : (
//                             svr.destination
//                           )
//                         }
//                         key={`${wip.num}-${pool.id}-${svr.id}`}
//                       />
//                     ))}
//                   </TreeNode>
//                 ))}
//               </TreeNode>
//             </Tree>
//           </div>
//         ))}
//       </div>
//     ))}
//   </div>
// );

const lbDict = {
  "round-robin": "Round Robin",
  "global-availability": "Global Availability"
};

const convertTime = time =>
  time
    .split("T")
    .join(" ")
    .substring(0, time.lastIndexOf(":00+"));

const RequestShortDesc = ({ record }) => {
  return (
    <div>
      {record.items.map(item => (
        <div key={item.id}>
          <span>
            <Icon type="calendar" /> {convertTime(item.impl_time)}
          </span>
          {Object.values(item.wideip).map(wip => (
            <div key={wip.name}>
              <span>
                <Icon type="fork" /> {lbDict[wip.loadbalance]}
              </span>
              <Tree showLine defaultExpandAll={true}>
                <TreeNode
                  title={
                    <div>
                      <Tag
                        color={colorTag[item.CRUD]}
                        style={{ cursor: "text" }}
                      >
                        {item.CRUD}
                      </Tag>
                      {wip.name}
                    </div>
                  }
                  key={wip.num}
                >
                  {Object.values(wip.pools).map(pool =>
                    pool.servers.length === 1 ? (
                      <TreeNode
                        title={
                          pool.servers[0].pub_destination ? (
                            <span>
                              {pool.servers[0].destination} (
                              <span style={{ color: "red" }}>
                                {pool.servers[0].pub_destination}
                              </span>
                              )
                            </span>
                          ) : (
                            pool.servers[0].destination
                          )
                        }
                        key={`${wip.num}-${pool.id}`}
                      />
                    ) : null
                  )}
                </TreeNode>
              </Tree>
            </div>
          ))}
        </div>
      ))}
    </div>
  );
};

export default RequestShortDesc;
