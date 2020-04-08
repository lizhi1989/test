import React from "react";
import { Tree, Icon } from "antd";

const TreeNode = Tree.TreeNode;

// const RequestShortDesc = ({ record }) => (
//   <Tree showLine defaultExpandAll={true}>
//     {record && (
//       <TreeNode title={<div>{record.name}</div>} key={record.num}>
//         {Object.values(record.gtm.svc).map(svc => (
//           <TreeNode title={svc.name} key={`0-${svc.id}`}>
//             {Object.values(svc.servers).map(svr => (
//               <TreeNode
//                 title={
//                   svr.translation ? (
//                     <span>
//                       {svr.translation} (
//                       <span style={{ color: "red" }}>{svr.destination}</span>)
//                     </span>
//                   ) : (
//                     svr.destination
//                   )
//                 }
//                 key={`0-${svc.id}-${svr.id}`}
//               />
//             ))}
//           </TreeNode>
//         ))}
//       </TreeNode>
//     )}
//   </Tree>
// );

const lbDict = {
  "round-robin": "Round Robin",
  "global-availability": "Global Availability"
};

const RequestShortDesc = ({ record }) => {
  return (
    <div>
      <span>
        <Icon type="fork" /> {lbDict[record.gtm.loadbalance]}
      </span>
      <Tree showLine defaultExpandAll={true}>
        {record && (
          <TreeNode title={<div>{record.name}</div>} key={record.num}>
            {Object.values(record.gtm.svc).map(svc => {
              return Object.keys(svc.servers).length === 1
                ? Object.values(svc.servers).map(svr => (
                    <TreeNode
                      title={
                        svr.translation ? (
                          <span>
                            {svr.translation} (
                            <span style={{ color: "red" }}>
                              {svr.destination}
                            </span>
                            )
                          </span>
                        ) : (
                          svr.destination
                        )
                      }
                      key={`0-${svc.id}`}
                    />
                  ))
                : null;
            })}
          </TreeNode>
        )}
      </Tree>
    </div>
  );
};

export default RequestShortDesc;
